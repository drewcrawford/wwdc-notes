[[Explore ShazamKit]]

# Building custom catalogs
Need to come up with ways to keep children engaged.  What if you could play a video on your apple tv and display questions at the right time?

1.  Create SHSignaturegenerator
2.  `installTap` and append buffer
3.  Converts audio to signature.  `addReferenceSignature`.

I'm going to build a catalog by adding a signature with associated metadata.  

Define the metadata using media items.  
# Matching audio
I included code to request microphone permission and setup the audio session.  

 Can be called multiple times with the same match?
 
 

# Content synchronization
Happens automatically I guess?

# Best practices
* Share custom catalogs
	* File or network
* Customize metadata
* Use matchStreamingBuffer where possible
* 