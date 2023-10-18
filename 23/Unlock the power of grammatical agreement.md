Discover how you can use automatic grammatical agreement in your apps and games to create inclusive and more natural-sounding expressions. We'll share best practices for working with Foundation, showcase examples in multiple languages, and demonstrate how to use these APIs to enhance the user experience for your apps. For an introduction to automatic grammatical agreement, watch “What's new in Foundation” from WWDC21.

Words matter.  Especialyl when you don't speak the language.

[[What's new in Foundation]] - grammatical agreement.  Today I'll introduce new features.

more correct, better sounding, and more inclusive.

# Grammatical agreement
we believe well-designed APIs can help ease linguistic complexity.

Last year, french-speaking users can select tehir term of address.  Software will change the wods to reflect that.

this year, we've added support to two new locales: portoguese and german.
# Dependency agreement
often, words need to change based on another word.  Let's walk through an example.

ex, consider size agreement with product type.  Since these words are separated, handling this word is nontrivial.

###  agreeWithConcept - 4:08
```swift
// Formatting the string

var options = AttributedString.LocalizationOptions()
options.concepts = [.localizedPhrase(food.localizedName)]

let size = AttributedString(localized: "small", options: options)
```

'concepts' - affect grammar of string but not localized like a variable.

`^[pequeno](agreeWithConcept: 1)`

on older devices, we simply ignore the attribute.

consider

```
^[Nuestro %@](inflect:true) esta hehco de %@
```

| differentiation | inflect   | agreeWithConcept | agreeWithArgument |
| --------------- | --------- | ---------------- | ----------------- |
| code changes    | no        | yes              | no                |
| proximity       | immediate | detached         | same string                  |

indicates that `hecho` whould agree with an argument elsewhere

```
^[Nuestro %@](inflect:true) esta ^[hecho](agreeWithArgument: 1) de %@
```

Notice that the sizes as well as the button are alreayd fixed for spanish.  Concept and inflect attributes.  

## Dependency agreement
* inflect words based on other worsd
* Use `agreeWithConcept` for detached words
* Use `agreeWithARgument` for word in the same string

# Inclusive language
When localizing UI, use gender-neutral language.  Using gendered language can feel moer personal though.

To represent that,a d a new property of type `termOfAddress` which is new this year.  We can set this to masculine, feminine, or neutral.  Since we want to refer to tony using he/him, we choose masculine.
###  Preferred terms of address - 8:45
```swift
// A person who is delivering the food order

struct DeliveryPerson {

    // The person's preferred name
    var name: String

    // An avatar for the delivery person
    var avatar: Image

    // The person's preferred terms of address. This list may contain more than
    // one option, we will use the first applicable one for the language that's
    // used in the UI.
    var preferredTermsOfAddress: [TermOfAddress]
}

// Formatting the message in Swift

var options = AttributedString.LocalizationOptions()
options.concepts = [.termsOfAddress(person.preferredTermsOfAddress)]

let message = AttributedString(localized:  "\(person.name) is on ^[their](referentConcept: 1) way.”, options: options)
```

value of 1 - reference should be first argument to options?

Notice how they/their are replaced with his/he.  Use the same localized string to refer t other people too.

feminine - refer to someone using she/her pronouns.  Using neutral to refer to someone using they/them pronouns.

`.localized` to specify language, and list all pronoun forms that you wish to use using the new morphology pronoun type.

each pronoun consists of the pronoun form in the target language, etc.  Refer to documentation for detailed instructions on constructing pronouns.
We use this technique to tailor the language used in iOS.

## demo

# Wrap up
* grammatical agreement
* dependency agreement
* inclusive language

I hope you love these new features, etc.


# Resources
* https://developer.apple.com/localization/
* https://developer.apple.com/documentation/foundation/nsinflectionrule
* https://developer.apple.com/documentation/foundation/nsmorphology
