Weather is even more critical.  Having accurate to forecasts is more important now than ever.  Powered by all-new apple weather service.  World class global weather forecast.

* high-eresolution weather models
* machine learning and prediction algorithms
* hyperlocal weather forecasts around the globe
* tons of data!
* without compromising user information
# Privacy
* location used only for weather forecasts
* no personally identifying information
* never shared or sold
# Available datasets
* Current weather.  Single point in time, many many conditions.
* Minute forecast.  Precipation for the next hour where available.
* Staring on the current hour, up to 240 hours.  Each hour has various datapoints.
* Daily forecast of 10 days.  Entire day, such as hi/lo temp, sunrise, etc.
* Weather alerts.  Severe warnings for the location.  Important information.
* Historical weather provides saved weather forecasts from the past so you can see trends.  Access via start/end date.  Lots of data.
# Requesting weather
Native framework, and REST APIs.

## weatherkit
#weatherkit

```swift
// Request the weather

import WeatherKit
import CoreLocation


let weatherService = WeatherService()

let syracuse = CLLocation(latitude: 43, longitude: -76)

let weather = try! await weatherService.weather(for: syracuse)

let temperature = weather.currentWeather.temperature

let uvIndex = weather.currentWeather.uvIndex
```

Sample project.  

Enable capability.  

Must display attribution for the data sources in my app.  First step is the attribution URL from `attribution.legalPageURL`.
Also need `combinedMark` for light/dark.

Note that apple weather mark and attribution link open in SFSafariVC.  

## REST API
Same rich weather data as the swift framework, on any platform.

```js
/* Request a token */
const tokenResponse = await fetch('https://example.com/token');
const token = await tokenResponse.text();

/* Get my weather object */
const url = "https://weatherkit.apple.com/1/weather/en-US/41.029/-74.642?dataSets=weatherAlerts&country=US"

const weatherResponse = await fetch(url, {
headers: {
"Authorization": token
}
});
const weather = await weatherResponse.json();

/* Check for active weather alerts */
const alerts = weather.weatherAlerts;
const detailsUrl = weather.weatherAlerts.detailsUrl;
```

Be sure to set the appropriate language for a localized response.  Lat/lon of location of interest.  Indicate the desired dataset.  You can request several csv.  country code is only required if you are requesting the weather alerts dataset.

### Auth
Create authentication key in developer portal.  WeatherKit requires token to validate authorization on each request.  Signed JWT with your private key.

* create JSON web token header
* Create JSON web token payload
* Return signed JSON Web token


# Publishing requirements

* required for both Swift and REST API
* Display active link to attribution
* Display apple weather logo
* Provide link to weather alert attribution (if displaying these alerts)

# Wrap up
* hpyerlocal forecasts powered by apple weather
* Swift framework for apple platforms
* REST API for anywhere
* download sample
* file feedback


* https://developer.apple.com/forums/tags/wwdc2022-10003
* https://developer.apple.com/forums/create/question?&tag1=356030&tag2=349030
* https://developer.apple.com/documentation/WeatherKit

