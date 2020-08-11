Fast Healthcare Interopoerability Resources

Health establishes a secure connection directly to the provider's API and downloads FHIR data directly.
This data is stored securely in HealthKit and allows aggregation of data.
Clinical data is neatly laid out in the health data.

# How your app can get this data?

1.  Get authorization from user
2.  User is guided through process in 3 screens.
	1.  General information about what granting access entails
	2.  Explanation text from plist is presented alongside a link to the privacy policy.  And yes, we will review your PP.
	3.  By default, only data already on the device will be shared.  So request authorization each time.  Sheet won't appear if there's nothing to authorize for.

[[Accessing Health Crecords with HealthKit - 18]]

# FHIR structure
Prevoiusly, you had to write your own Swift Codable if you wanted to work with native format.
Straightforward if you only want to work with certain elements.

Soon, you start to realize the depth, complexity, and number of FHIR resources.

# FHIR models
Open source package on GitHub.
https://github.com/apple/FHIRModels
 * open source Swift package
 * Supports FHIR DSTU2, R4, latest build
 * Data models for all resources
 * Validation of date/time, codes, value\[x] and much more
 * Apple mantained

```swift
// Use with Health Records FHIR data from HealthKit
import HealthKit
import ModelsDSTU2

// Grab HKClinicalRecord from HealthKit API
let clinicalRecord: HKClinicalRecord
let resource = clinicalRecord.fhirResource!

// Print the prescription note
let decoder = JSONDecoder()
let prescription = try decoder.decode(MedicationOrder.self, from: resource.data)
print("\(prescription.note)")
```

```swift
// Make using "TimingRepeat" period dates easier by writing an extension
extension TimingRepeat {
    var periodDisplayString: String? {
        if case .period(let period) = bounds {
            return "\(period.start) - \(period.end)"
        }
        return nil
    }
}

// Collect all dosage instructions on medication prescriptions
let instructions: [String] = prescription.dosageInstruction?.map { dosage in
    guard let period = dosage.timing?.repeat?.periodDisplayString else {
        return "\(dosage.text)"
    }
    return "\(period): \(dosage.text)"
}
```

# Releases
* DSTU-1
* DSTU-2
* STU-3
* R4
* R5

```swift
// Supporting multiple releases
import ModelsDSTU2
import ModelsR4

let decoder = JSONDecoder()
let release: FHIRRelease
let data: Data

let note: String? = nil
switch release {
case .dstu2:
    let model = try decoder.decode(ModelsDSTU2.MedicationOrder.self, from: data)
    note = model.note?.value?.string
case .r4:
    let model = try decoder.decode(ModelsR4.MedicationRequest.self, from: data)
    note = model.note?.compactMap({ $0.text.value?.string }).joined(separator: "\n")
default:
    note = "Unsupported FHIR release \(release)"
}
```

# Wrap up
Helps you deal with the complexity of FHIR resources and releases
Use the package when working with clinical data obtained from HealthKit
Take a look at "What's new in CareKit" to see how you can use FHIR with CareKit
Enhance your own, independent FHIR apps

* clone the library and add it to your project
* File issues, even if you just want to provide feedback
* Get our sample app, which shows how to integrate
* Learn more about FHIR at chat.fhir.org

