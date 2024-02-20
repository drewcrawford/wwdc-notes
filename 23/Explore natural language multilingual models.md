Learn how to create custom Natural Language models for text classification and word tagging using multilingual, transformer-based embeddings. We'll show you how to train with less data and support up to 27 different languages across three scripts. Find out how to use these embeddings to fine-tune complex models trained in PyTorch and TensorFlow. For more on Natural Language, check out "Make apps smarter with Natural Language” from WWDC20.

# Evolution of NLP

Start off with text, convert to features. (vector?)
NLP model
produces output text.

in 2013, things moved to word embeddings
2016 -> contextual embeddings based on CNN/LSTM, etc.
2019 -> transformers.

What is an embedding?  A map from words in al anguage to vectors in an abstract vector space.  Such that words with similar meanings are close together in vector space.  To incporate linguistic knowledge.

Static embedding -> look up in table.
Contextual embeddings -> each word in a sentence is mapped to a different vector depending on its use in a sentence.  

w/o requiring huge amounts of task-specific training data.  CreateML supports ELMO models (LSTM).  Outputs combined to produce embedding vector.


# NL understanding

Models for two different tasks.  Classification and word tagging.

classification -> describe input text with classes.
word tagging -> label words ex parts of speech.

Related videos

[[Make Apps Smarter with Natural Language]]
[[Advances in Natural Language Framework - 19]]

# Multilingual embeddings

Transformer-based contextual embeddings (BERT).
Trained on large amounts of text.  MOdel is given a sentence with one word masked out and asked to suggets a word.  

Transformers at their heart are based on attention mechanism.  Specifically multi-headed self attention.  Take into account different portions of text with different weights.

Multi-headed self-attention is wrapped up wtih various layers and repeated many times.  Take advantage of large amounts of textual data.

Each model is trained on data from multiple languages
One model per group of related scripts
Take advantage of similarties between languages

We support 27 different languages.  3 separate models.  For groups of languages.

Latin script -> 20 languages
Cyrillic script -> 4 languages
CJK - chinese, japanese, korean

training data -> BERT -> training -> model

Recommend using training data for each language you're interested in.

NLContextualEmbedding APIs.
Find embedding for speicfic languages or scripts.

Embedding models rely on assets that rae downloaded as needed.
Use asset APIs
* create embedding object
* check if assets are present
* etc.




# Advanced applications

translate to get different images

# Wrap up
train models using create ML
Use the new multilingual BERT embeddings
use BERT embeddings within your own models.



# Resources
* https://developer.apple.com/documentation/naturallanguage
