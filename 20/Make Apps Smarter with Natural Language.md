Language is a code system that helps humans solves problems through communication.

Language is an IR between concepts (thoughts) and expressions (speech, writing ,etc.)

Also works in reverse.

Language has been replaced with NLP.  Intermediate representation that helps machine translate concepts into symbols, and also sassimilate content into concepts.  But what does on-device NLP at apple look like?

Until 2017, `NSLinguisticTagger`.
In 2018, Natural Language framework.  Tightly integrating Natural Language Frmework with CoreML/CreateML.

**NSLinguisticTagger has been marked for deprecation.**

# Natural language framework
* fundamental text processing
* text embeddings
* custom models

## Fundamental text processing

* language identification
* tokenization
* part of speech
* lemmatization
* named entity recognition

[[Advances in Natural Language Framework - 19]]
[[Introducing Natural Language Framework - 18]]

Input: text
output: hypothesis / prediction.

This year, we have a brand new API for confidence.  In addition to the existing output.

```swift
import NaturalLanguage 

let tagger = NLTagger(tagSchemes: [.nameType])
let str = "Tim Cook is very popular in Spain."
let strRange = Range(uncheckedBounds: (str.startIndex, str.endIndex))
tagger.string = str
tagger.setLanguage(.english, range: strRange)
 
tagger.enumerateTags(in: str.startIndex..<str.endIndex, unit: .word, scheme: .nameType, options: .omitWhitespace) { (tag, tokenRange) -> Bool in
    let (hypotheses, _) = tagger.tagHypotheses(at: tokenRange.lowerBound, unit: .word, 
                                               scheme: .nameType, maximumCount: 1)
    print(hypotheses)
    return true
}
```

Consider a false positive for NER.  e.g. "Do Not Disturb While Driving" is not an organization name.  This scores pretty low, so you can filter out by confidence score.

* Avoid heuristic hard coding of threshold values
* Calibrate on representative data
* Consider setting threshold per-class

## Text embeddings
Text corpus: a collection of documents which are comprised of paragraphs, sentences, phrases and words.

We tokenize into a bag of words.

"One-hot encoding".  Represented by a bit vector which has 1 bit on, and the rest of the bits off.

When we use word embeddings, we get a vector representation of words.  Where words that are similar are clustered together, and words that are dissimilar are clustered away.

Each word gets a real-value vector of D dimensions (columns), where they're floating points.


### Static Word Embeddings

"I want a burger from a fast *food* joint" -> extract embedding.

We precompute the embeddings in a LUT.  Stored on device in an efficient manner.  So we simply look in the LUT.

"It is *food* for thought".  Here the connotation is different.

### Dynamic word embeddings
We pass every sentence through a NN.  We get a dynamic embedding for every word in that sequence.  Now we get different vectors per context.

Static word embeddings in a variety of languages and platforms

[[Advances in Natural Language Framework - 19]]

We support custom word embeddings where you can train your own embeddings in fasttext, word2vec, glove, etc.  You can then bring these into your app.

How to use?

```swift
import NaturalLanguage

if let embedding = NLEmbedding.wordEmbedding(for: .english) {
    let word = "bicycle"
    
    if let vector = embedding.vector(for: word) {
        print(vector)
    }
    
    let dist = embedding.distance(between:word, and: "motorcycle")
    print(dist)
    
    embedding.enumerateNeighbors(for: word, maximumCount: 5) { neighbor, distance in
        print("\(neighbor): \(distance.description)")
        return true
    }
}
```

#### example
Let's say you want to do search through a FAQ.

"Do you deliver to Cupertino?" -> get word embeddings for each word, then take the average of each word.  Can precompute the embeddings for every FAQ in your database.  So at runtime, you find the question that is closest to the input query vector and you pick that question.

Problems?
* word coverage.  If you have an input query that does not appear in the LUT, you will lose information.
* Noisy process
* Loses compositional knowledge.  "Do you deliver from cuptertino to san jose?" We lose context such as 'from' and 'to'.

### sentence embedding
Now can pass a sentence and encodes this information into a finite dimensional vector.  Currently, 512 dimensions.

Intuitively, you can think of this as starting from a text corpus, tokenize into sentence level, now we have "sentence vectors".  Sentences that are conceptually similar are clusted together, and sentences that are dissimilar are far away.

Chart of how this NN actually works.


```swift
import NaturalLanguage

if let embedding = NLEmbedding.sentenceEmbedding(for: .english) {
    let sentence = "This is a sentence."
    
    if let vector = sentenceEmbedding.vector(for: sentence) {
        print(vector)
    }
    
    let dist = sentenceEmbedding.distance(between: sentence, and: "That is a sentence.")
    print(dist)
}
```

```swift
func answerKey(for string: String) -> String? {
        guard let embedding = NLEmbedding.sentenceEmbedding(for: .english) else { return nil }
        guard let queryVector = embedding.vector(for: string) else { return nil }

        var answerKey: String? = nil
        var answerDistance = 2.0

        for (key, vectors) in self.faqEmbeddings {
            for (vector) in vectors {
                let distance = self.cosineDistance(vector, queryVector)
                if (distance < answerDistance) {
                    answerDistance = distance
                    answerKey = key
                }
            }
        }

        return answerKey
    }
```

If you're searching over a large number of results, you may need a custom embedding rather than doing a linear scan.

Key: strings from which we can readily determine which poem we were looking at
Values: the sentence vector.  Can do efficient nearest neighbor search without having to do the whole thing

```swift
import NaturalLanguage
import CreateML


let embedding = try MLWordEmbedding(dictionary: sentenceVectors)

try embedding.write(to: URL(fileURLWithPath: "/tmp/Verse.mlmodel"))
```

What comes out here is a CoreML model

```swift
func answerKeyCustom(for string: String) -> String? {
        guard let embedding = NLEmbedding.sentenceEmbedding(for: .english) else { return nil }
        guard let queryVector = embedding.vector(for: string) else { return nil }

        guard let (nearestLineKey, _) = self.customEmbedding.neighbors(for: queryVector, maximumCount: 1).first else { return nil }

        return self.poemKeyFromLineKey(nearestLineKey)
    }
```

### Clustering
If all the text comes in from the user.  Let's say you have messages, reviews, problem reports, etc.  Can take sentence embeddings, calculate a vector, and use standard clustering algorithms to find groups.

Available on a number of different languages and platforms.

* Use on natural text
* Do not remove stop words
* Break long text into paragraphs (sentences?)
* Use custom embedding for large collection\

## custom models
NLP tasks
* text classification -> take a piece of text and supply a label
* Word tagging -> take a sequence of words in a sentence and supply a label

CreateML task

[[Advances in Natural Language Framework - 19]]

Note that we get transfer learning from the word embeddings, because we already know what words mean.  So you don't need as much data.

### word tagging
Can potentially use word tagging to divide a sentence up into phrases.
Take a sentence and extract important pieces of information from it.
e.g. slot parsing.  Travel from, travel to, etc.

Potentially label parts of sentence as "food" or "city", etc.  

Obviously you could just have a list of keywords.  `NLGazeteer` has an efficient implementation.

But the problem is that in general, you're not going to be able to list all potential values.  As soon as you encounter some piece of text you haven't thought of, this search is not going to help you.

Also, this doesn't take into account meaning of words or context.  Word tagger can solve these problems.

Can combine word tagger and `NLGazeteer` for additional accuracy.

Where do I start?

* define labels
* collect sentences
* train

What *sushi* restaurants do you deliver from in *Cupertino*
Use "neutral" to mean the word is "other"
use "food" for sushi
use from_city for Cupertino

Note that we distinguish between "city" and "from_city".  The word tagger can take advantage of context to distinguish between these two.

```swift
import CreateML

let modelParameters = MLWordTagger.ModelParameters(algorithm: .crf(revision: 1))
```


[[Introducing Natural Language Framework - 18]]

New this year, we're applying the power of transfer learning to word tagging.

(code snippet not provided, but pass osmething else instead of 'crf')

Using custom tagger
```swift
func findTags(for string: String) {
        let model = try! NLModel(contentsOf: Bundle.main.url(forResource: "Nosh", withExtension: "mlmodelc")!)
        let tagger = NLTagger(tagSchemes: [NoshTags])

        tagger.setModels([model], forTagScheme: NoshTags)
        tagger.string = string

        tagger.enumerateTags(in: string.startIndex..<string.endIndex, unit: .word, scheme: NoshTags, options: .omitWhitespace) { (tag, tokenRange) -> Bool in

            let name = String(string[tokenRange])

            switch tag {
                case NoshTagRestaurant:
                    self.noteRestaurant(name)
                case NoshTagFood:
                    self.noteFood(name)
                case NoshTagFromCity:
                    self.noteFromCity(name)
                case NoshTagToCity:
                    self.noteToCity(name)
                default:
                    break
            }
            return true
        }
    }
```
Supported for same languages for static embeddings etc.

### Recommendations
* compare with CRF for english.  CRF pays attention to syntatic features which are useful for many applications.  However if you do not know the distribution, transfer learning provides better generalization.
* Requires more data for word tragging than text classification.  

# Wrap up
NSLinguisticTagger is deprecated, move to NaturalLanguage
Confidence socres for APIs
Sentence embeddings
Transfer learning for word tagging


