#sms

These allow you to create message filter extensions which help people categorize incoming SMS Messages from unknown senders

# SMS message filters
In many countries, SMS is used by businesses, etc., to notify customers about transactions, marketing campaigns, alerts, reminders, etc.

Cluttered inbox.  Tough to find personal messages in here.

iOS does provide an option to filter messages from unknown senders.  But if you receive several messages each day, even the unknown sender folder will quickly get filled with unread messages.  

Secure, sandbox-based extension model that allows you to further classify messages from unknown senders.  Find and install SMS filter apps.  Settings->Messages->Unknown & Spam, and turn on Filter Unknown Senders.  Select your SMS filter of choice.  Here we instaleld two apps.

Only one filter can be active at a time.

In iOS 14 and later, new folders will appear in messages for transactions, promotions, and junk.  These folders hepl people organize and find messages that are most relevant to them.  Regardless of which filter is chosen, messages provides the same classification structure.


# Enhancements in iOS 16
New sub-categoreis for transactions and promotions

Further refine incoming messages and provide an even better experience.  ex, in markets like india, it's common to receive a large number of messages related to financial transactions.  These include activities, bank account, alerts, credit card spending, etc.

configuration.  Users select your message filter in settings.  This triggers a new API introduced in iOS 16, to request the capabilities supported by your filter.  You can now respond with a list of supported categories and subcategories.  In this example, we report supoprting 3 subcategories.

Every time a message is received from unknown sender, iOS queries your filter to determine which category and subcategory it belongs to.  Filters must respond with a caetgory declared in configuration phase.

# Build a message filter
Start by creating a new message filter extension target, the message filter extension target appears as one of the options when you create a new target.  When you go to template selection.  Select the message filter extension.

Name, etcc

```swift
func handle(_ capabilitiesRequest: ILMessageFilterCapabilitiesQueryRequest, context: ILMessageFilterExtensionContext, completion: @escaping (ILMessageFilterCapabilitiesQueryResponse) -> Void) {
    let response = ILMessageFilterCapabilitiesQueryResponse()
    // choose up to five sub-categories supported by the filter
    response.transactionalSubActions = [.transactionalFinance,
                                        .transactionalOrders,
                                        .transactionalHealth]
    response.promotionalSubActions   = [.promotionalCoupons,
                                        .promotionalOffers]
    completion(response)
}
```

Can specify up to 5 subactions.  

messages now shows these subcategories we selected.

```swift
func handle(_ queryRequest: ILMessageFilterQueryRequest, context: ILMessageFilterExtensionContext, completion: @escaping (ILMessageFilterQueryResponse) -> Void) {
    guard let message = queryRequest.messageBody else { return }
    let response = ILMessageFilterQueryResponse()
    switch(message) {
    case _ where message.contains("debited"):
        response.filterAction = .transaction
        response.filterSubAction = .transactionalFinance
        break
    case _ where message.contains("coupon"):
        response.filterAction = .promotion
        response.filterSubAction = .promotionalCoupons
        break
     // update other cases
    }
    completion(response)
}
```

If you return an incorrect combination for filter actions/subactions, iOS will discard the subaction and only honor the action.  For example, if you return the action transaction and subaction coupons, then the message will only go to the .. transaction folder.

in iOS 16, you can choose subcategories that are the best fit for your demographic.  

Use these subcategories to provide a different experience for your users.

# Apple SMS filter for india

Apple provides a filter for india. We updated it.  Apple filter in india now supports additional subfolders.  Including ifnance, orders, reminders, under transactions.  

Finance => banks, etc.
orders => food transactions, etc.
todos => reminders

# Wrap up
* SMS message filters
* Enhancements in iOS 16

* https://developer.apple.com/forums/tags/wwdc2022-110341
* https://developer.apple.com/forums/create/question?&tag1=163&tag2=453030
* https://developer.apple.com/documentation/sms_and_call_reporting/sms_and_mms_message_filtering
