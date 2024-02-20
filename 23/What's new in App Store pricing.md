Discover the latest updates to App Store pricing capabilities and tools. Learn how you can manage pricing for your apps and in-app purchases within App Store Connect and the App Store Connect API, how to set pricing by region, and more.

Largest set of upgrades to app store capability since first launch.

# Enhanced global pricing

44 currencies, 175 regions.

* Expanded price points (900)
	* By default, you get access to 800, and by request you can get 100 more.
* 44 currencies
* In addition, we have introduced a pricing tool to configure base region for base price from any of 175 regions.  Streamlined global pricing.
* Important to note that the base price will not be automatically adjusted.  it's the base for other prices.
* But we adjust other prices as currencies and tax situations change.

## Set up global pricing

You can choose different prices for individual countries/regions.  But youw ill not benefit from automatic repricing.  You're responsible for reflecting those changes in your pricing.


In this example, let's schedule a global price change.

* effective date
* base price
## API examples

AppPriceSchedules
* base territory
* ManualPrices
* AutomaticPrices

LInked to AppPricePoint, linked to Territory.  

need to make 3 API calls to get entire price schedule.

1.  Use GET appPriceSchedules/appID/baseTerritory
2. GET manualPrices?include=appPricePoint,territory
3. GET automaticPricnes?include=appPricePoint,territory

Update price schedule:

1.  Get all price points of base country or region (`/appPricePoints?filter[territory]=USA`)
2. Preview comparable price points of base price (`equalizations?include=territory`)
3. Update the price schedule (`POST appPriceSchedules`  app ID, baseTerritory, manualPrices (at least base country/region))

You'll know you can use any temporary ID, just give `${something}`.  

startDate: null means immediately.  So test this against non-live apps please.

# Pricing by region

More control over your pricing at the region level.  Advanced pricing, manually set custom prices and temporary prices with any price point on a per-region basis.

Let's start with temporary prices.  Short period of time.  Let me demonstrate with example.

## Custom prices
On a permanent basis. 





## Demos

You can also do this via API.  Important to remember that temporary prices and custom prices are manual prices.  


I will use `POST /appPriceSchedules`.  Replace manualPrices with base price AND new temporary price in some other territory.  

Managing pricing for your products will be much easier now.

# Pricing tips

* explore expanded price points
* Review new base price capability
* Carefully review changes to ensure pricing calculations are set as expected
* Adopt the latest APIs.  **We intend to retire old APIs later this year.**



