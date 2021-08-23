A new way to create audio drivers with DriverKit.

#driverkit

Moving out of the kernel and into userspace

[[Modernize PCI and SCSI drivers with driverkit]]
[[System Extensions and DriverKit - 19]]

Two components are still needed to implementa n audio driver.
1.  Audio server plug-in
2.  driverkit extension

This complicates development.  Starting in monterey, you only need a dex, no more plugin.

# AudioDriverKit
* Familiar C++ DriverKit API
* Single process
* Bundled with an App
* Loaded dynamically

# Architecture
Private user client that will be used for all communication between coreaudio and audio dext.

This isn't intended to be used directly.  No plugin or custom user client required.

Your app can open a custom user client to communicate with the application directly if needed.
# Prerequisites
`com.apple.developer.driverkit`
`com.apple.developer.driverkit.allow-any-userclient-access`

`com.apple.developer.driverkit.transport.pci`

If you haven't requested these

developer.apple.com/contact/request/sytem-extension/

Entitlements not needed for sample usecase.  

Info.plist: Various settings.  IOUserAudioDriverUserClientProperties dict.
IOClass =>
IOUserClass =>

See `<AudioDriverKit/AudioDriverTypes.h>` for more information
# Initialization
Sublclass `IOUserAudioDriver` and override virtual methods
Subclass any objects such as `IOUserAudioDevice` that need custom behavior
Configure audio objects and add to `IOUserAudiodriver`

Device:
* OSTimerDispatchSource
	* IOUserAudioStream
* OSAction
	* IOUserAudioVolumeLevelControl
* Tone generator
	* IOUserAudioCustomProperty

```cpp
// SimpleAudioDriver example, subclass of IOUserAudioDriver

class SimpleAudioDriver: public IOUserAudioDriver
{
public:
	virtual bool init() override;
	virtual void free() override;
	
	virtual kern_return_t Start(IOService* provider) override;	
	virtual kern_return_t Stop(IOService* provider) override;
	virtual kern_return_t NewUserClient(uint32_t in_type,
										 IOUserClient** out_user_client) override;
	
	virtual kern_return_t StartDevice(IOUserAudioObjectID in_object_id,
									   IOUserAudioStartStopFlags in_flags) override;
	
	virtual kern_return_t StopDevice(IOUserAudioObjectID in_object_id,
									  IOUserAudioStartStopFlags in_flags) override;
```
```cpp
// SimpleAudioDriver example override of IOService::NewUserClient
kern_return_t SimpleAudioDriver::NewUserClient_Impl(uint32_t in_type,
													 IOUserClient** out_user_client)
{
	kern_return_t error = kIOReturnSuccess;
	//	Have the super class create the IOUserAudioDriverUserClient object if the type is
	//	kIOUserAudioDriverUserClientType
	if (in_type == kIOUserAudioDriverUserClientType)
	{
		error = super::NewUserClient(in_type, out_user_client, SUPERDISPATCH);
	}
	else
	{
		IOService* user_client_service = nullptr;
		error = Create(this, "SimpleAudioDriverUserClientProperties", &user_client_service);
		FailIfError(error, , Failure, "failed to create the SimpleAudioDriver user-client");
		*out_user_client = OSDynamicCast(IOUserClient, user_client_service);
	}
	return error;
}
```

```cpp
// SimpleAudioDevice::init, set device sample rates and create IOUserAudioStream object
...
    SetAvailableSampleRates(sample_rates, 2); 
    SetSampleRate(kSampleRate_1);

    //	Create the IOBufferMemoryDescriptor ring buffer for the input stream
    OSSharedPtr<IOBufferMemoryDescriptor> io_ring_buffer;
    const auto buffer_size_bytes = static_cast<uint32_t>(in_zero_timestamp_period *  
        sizeof(uint16_t) * input_channels_per_frame);
    IOBufferMemoryDescriptor::Create(kIOMemoryDirectionInOut, buffer_size_bytes, 0, 
       io_ring_buffer.attach());
	
    //	Create input stream object and pass in the IO ring buffer memory descriptor
    ivars->m_input_stream = IOUserAudioStream::Create(in_driver, 
                                                      IOUserAudioStreamDirection::Input, 
                                                      io_ring_buffer.get());
...
```

# Create audio objects
* Subclass IOUserAudioDevice
* In IOUserAudioDevice::init, create objects

```cpp
// SimpleAudioDevice::init, set device sample rates and create IOUserAudioStream object
...
    SetAvailableSampleRates(sample_rates, 2); 
    SetSampleRate(kSampleRate_1);

    //	Create the IOBufferMemoryDescriptor ring buffer for the input stream
    OSSharedPtr<IOBufferMemoryDescriptor> io_ring_buffer;
    const auto buffer_size_bytes = static_cast<uint32_t>(in_zero_timestamp_period *  
        sizeof(uint16_t) * input_channels_per_frame);
    IOBufferMemoryDescriptor::Create(kIOMemoryDirectionInOut, buffer_size_bytes, 0, 
       io_ring_buffer.attach());
	
    //	Create input stream object and pass in the IO ring buffer memory descriptor
    ivars->m_input_stream = IOUserAudioStream::Create(in_driver, 
                                                      IOUserAudioStreamDirection::Input, 
                                                      io_ring_buffer.get());
...
```

```cpp
// SimpleAudioDevice::init continued
    IOUserAudioStreamBasicDescription input_stream_formats[2] = {
            kSampleRate_1, IOUserAudioFormatID::LinearPCM,
            static_cast<IOUserAudioFormatFlags>(
                    IOUserAudioFormatFlags::FormatFlagIsSignedInteger | 
                    IOUserAudioFormatFlags::FormatFlagsNativeEndian),
            static_cast<uint32_t>(sizeof(int16_t)*input_channels_per_frame),
            1,
            static_cast<uint32_t>(sizeof(int16_t)*input_channels_per_frame),
            static_cast<uint32_t>(input_channels_per_frame),
            16
		    },
        ...
    }

    ivars->m_input_stream->SetAvailableStreamFormats(input_stream_formats, 2);
    ivars->m_input_stream_format = input_stream_formats[0];
    ivars->m_input_stream->SetCurrentStreamFormat(&ivars->m_input_stream_format);
	
    error = AddStream(ivars->m_input_stream.get());
```

```cpp
//	Create volume control object for the input stream.
    ivars->m_input_volume_control = IOUserAudioLevelControl::Create(in_driver,
        true, -6.0, {-96.0, 0.0},
        IOUserAudioObjectPropertyElementMain,
        IOUserAudioObjectPropertyScope::Input,
        IOUserAudioClassID::VolumeControl);

    //	Add volume control to device
    error = AddControl(ivars->m_input_volume_control.get());
```

Custom properties
```cpp
// SimpleAudioDevice::init, Create custom property

IOUserAudioObjectPropertyAddress prop_addr = {
    kSimpleAudioDriverCustomPropertySelector,
    IOUserAudioObjectPropertyScope::Global,
    IOUserAudioObjectPropertyElementMain
};

custom_property = IOUserAudioCustomProperty::Create(in_driver, prop_addr, true,
    IOUserAudioCustomPropertyDataType::String,
    IOUserAudioCustomPropertyDataType::String);

qualifier = OSSharedPtr(
    OSString::withCString(kSimpleAudioDriverCustomPropertyQualifier0), OSNoRetain);
data = OSSharedPtr(
    OSString::withCString(kSimpleAudioDriverCustomPropertyDataValue0), OSNoRetain);
custom_property->SetQualifierAndDataValue(qualifier.get(), data.get());

AddCustomProperty(custom_property.get());
```


# IO path and timestamps
Use `GetIOMemoryDescriptor` to get the IO buffer used by the IOUserAudioStream.

CoreAudio HAL will read and write to the IO buffer used by the stream
Use the stream IO buffer for DMA to the hardware
User IOUserAudioClockDevice UpdateCurrentZeroTimestamp and `GetCurrentZeroTimestamp`
Updating the zero timestamp should use the most current timestamp tracked by the hardware

```cpp
// SimpleAudioDevice example subclass of IOUserAudioDevice

class SimpleAudioDevice: public IOUserAudioDevice
{
...
    virtual kern_return_t StartIO(IOUserAudioStartStopFlags in_flags) final LOCALONLY;
    virtual kern_return_t StopIO(IOUserAudioStartStopFlags in_flags) final LOCALONLY;

private:
    kern_return_t StartTimers() LOCALONLY;
    void StopTimers() LOCALONLY;
    void UpdateTimers() LOCALONLY;
    virtual void ZtsTimerOccurred(OSAction* action,
								   uint64_t time) TYPE(IOTimerDispatchSource::TimerOccurred);
    virtual void ToneTimerOccurred(OSAction* action,
									uint64_t time) TYPE(IOTimerDispatchSource::TimerOccurred);
    void GenerateToneForInput(size_t in_frame_size) LOCALONLY;
}
```

```cpp
// StartIO 
kern_return_t SimpleAudioDevice::StartIO(IOUserAudioStartStopFlags in_flags)
{
    __block kern_return_t error = kIOReturnSuccess;
    __block OSSharedPtr<IOMemoryDescriptor> input_iomd;
    ivars->m_work_queue->DispatchSync(^(){
		//	Tell IOUserAudioObject base class to start IO for the device
        error = super::StartIO(in_flags);
        if (error == kIOReturnSuccess)
        {
            // Get stream IOMemoryDescriptor, create mapping and store to ivars	
            input_iomd = ivars->m_input_stream->GetIOMemoryDescriptor();
            input_iomd->CreateMapping(0, 0, 0, 0, 0, ivars->m_input_memory_map.attach());

            // Start timers to send timestamps and generate sine tone on the stream buffer	
            StartTimers();
        }
    });
    return error;
}
```
```cpp
kern_return_t SimpleAudioDevice::StartTimers()
{
...
	//	clear the device's timestamps
	UpdateCurrentZeroTimestamp(0, 0);
	auto current_time = mach_absolute_time();
	auto wake_time = current_time + ivars->m_zts_host_ticks_per_buffer;
	
	//	start the timer, the first time stamp will be taken when it goes off
	ivars->m_zts_timer_event_source->WakeAtTime(kIOTimerClockMachAbsoluteTime,
												 wake_time,
												 0);
	ivars->m_zts_timer_event_source->SetEnable(true);
...
}
```

```cpp
void SimpleAudioDevice::ZtsTimerOccurred_Impl(OSAction* action, uint64_t time)
{
...
	GetCurrentZeroTimestamp(&current_sample_time, &current_host_time);
	auto host_ticks_per_buffer = ivars->m_zts_host_ticks_per_buffer;
	if (current_host_time != 0) {
		current_sample_time += GetZeroTimestampPeriod();
		current_host_time += host_ticks_per_buffer;
	}
	else {
		current_sample_time = 0;
		current_host_time = time;
	}
	// Update the device with the current timestamp
	UpdateCurrentZeroTimestamp(current_sample_time, current_host_time);

	//	set the timer to go off in one buffer
	ivars->m_zts_timer_event_source->WakeAtTime(kIOTimerClockMachAbsoluteTime,
												current_host_time + host_ticks_per_buffer, 0);
}
```

```cpp
void SimpleAudioDevice::GenerateToneForInput(size_t in_frame_size) 
{
	// Fill out the input buffer with a sine tone
	if (ivars->m_input_memory_map)
	{
		//	Get the pointer to the IO buffer and use stream format information
		//	to get buffer length
		const auto& format = ivars->m_input_stream_format;
		auto buffer_length = ivars->m_input_memory_map->GetLength() / 
            (format.mBytesPerFrame / format.mChannelsPerFrame);
		auto num_samples = in_frame_size;
		auto buffer = reinterpret_cast<int16_t*>(ivars->m_input_memory_map->GetAddress() +  
            ivars->m_input_memory_map->GetOffset());
...	
}
```

```cpp
void SimpleAudioDevice::GenerateToneForInput(size_t in_frame_size) 
{
...
	auto input_volume_level = ivars->m_input_volume_control->GetScalarValue();

	for (size_t i = 0; i < num_samples; i++)
	{
		float float_value = input_volume_level * 
    sin(2.0 * M_PI * frequency * 
       static_cast<double>(ivars->m_tone_sample_index) / format.mSampleRate);

		int16_t integer_value = FloatToInt16(float_value);
		for (auto ch_index = 0; ch_index < format.mChannelsPerFrame; ch_index++)
		{
			auto buffer_index =
                (format.mChannelsPerFrame * ivars->m_tone_sample_index + ch_index) %         
                buffer_length;
			buffer[buffer_index] = integer_value;
		}
		ivars->m_tone_sample_index += 1;
	}
}
```


# Configuration changes
```cpp
// IOUserAudioClockDevice.h and IOUserAudioDevice.h

kern_return_t RequestDeviceConfigurationChange(uint64_t in_change_action,
											    OSObject* in_change_info);

virtual kern_return_t PerformDeviceConfigurationChange(uint64_t in_change_action,
											            OSObject* in_change_info);

virtual kern_return_t AbortDeviceConfigurationChange(uint64_t change_action,
													  OSObject* in_change_info);
												
```

A common scenario is updating the current sample rate.  `request` ensures there is no data inflight when changes are actually made.

1.  Request
2.  HAL notify any listeneres that change will begin
3.  IO is stopped on device
4.  current device state is captured
5.  performDeviceconfigurationChange() is called
6.  Driver can change state
7.  New state is captured
8.  Notify listeners about changes to device state
9.  If IO was running, StartDevice() will be called.
10.  End config change notification

```cpp
kern_return_t SimpleAudioDriver::HandleTestConfigChange()
{
	auto change_info = OSSharedPtr(OSString::withCString("Toggle Sample Rate"), OSNoRetain);
	return ivars->m_simple_audio_device->RequestDeviceConfigurationChange(
        k_custom_config_change_action, change_info.get());
}

class SimpleAudioDevice: public IOUserAudioDevice
{
...
	virtual kern_return_t PerformDeviceConfigurationChange(uint64_t change_action,
												    OSObject* in_change_info) final LOCALONLY;
}
```


```cpp
kern_return_t SimpleAudioDriver::HandleTestConfigChange()
{
	auto change_info = OSSharedPtr(OSString::withCString("Toggle Sample Rate"), OSNoRetain);
	return ivars->m_simple_audio_device->RequestDeviceConfigurationChange(
        k_custom_config_change_action, change_info.get());
}

class SimpleAudioDevice: public IOUserAudioDevice
{
...
	virtual kern_return_t PerformDeviceConfigurationChange(uint64_t change_action,
												    OSObject* in_change_info) final LOCALONLY;
}
```

```cpp
// In SimpleAudioDevice::PerformDeviceConfigurationChange
	kern_return_t ret = kIOReturnSuccess;
	switch (change_action) {
		case k_custom_config_change_action: {
			if (in_change_info)	{
				auto change_info_string = OSDynamicCast(OSString, in_change_info);
				DebugMsg("%s", change_info_string->getCStringNoCopy());
			}

			double rate_to_set = static_cast<uint64_t>(GetSampleRate()) != 
                static_cast<uint64_t>(kSampleRate_1) ? kSampleRate_1 : kSampleRate_2;
			ret = SetSampleRate(rate_to_set);
			if (ret == kIOReturnSuccess) {
				// Update stream formats with the new rate
				ret = ivars->m_input_stream->DeviceSampleRateChanged(rate_to_set);
			}
		}
			break;
			
		default:
			ret = super::PerformDeviceConfigurationChange(change_action, in_change_info);
			break;
	}
	
	// Update the cached format:
	ivars->m_input_stream_format = ivars->m_input_stream->GetCurrentStreamFormat();
	
	return ret;
}
```

To get rid of the driver extension, simply delete the application!

# Wrap up
* Audio server plugin + driverkit
* Intorduced the new AudioDriverKit framework
* Wrote an audio driver extension using AudioDriverKit

* Download latest DriverKit SDK
* Adopt AudioDriverKit for suported hardware device families
* Provide feedback through Feedback Assistant
* 