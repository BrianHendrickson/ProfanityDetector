# ProfanityDetector

This is a C# (.NET Standard 2.1) library for detecting profanities within a text string. The profanity list was compiled from lists from the internet that are allegedly used by social media sites for detecting profanities. This is useful if you want to detect anything naughty in some text and have those words reported.

_The profanity list contains swearing, sexual acts, ratial slurs, sexist slurs and anything else that you can imagine._
_If you are easily offended, then **DO NOT** open the file called ProfanityList.cs_

In this readme I will cover the following:

* Using the Library via Nuget
* Basic Usage
* The Scunthorpe Problem
* Whitelisting
* Adding and Removing Profanties
* Replacing the Profanitiy List
* Frequently Asked Questions


# Using the Library via Nuget

If you do not wish to download or clone this repository, then you can consume the profanity detector via [Nuget](https://www.nuget.org/packages/Profanity.Detector/).

To install via the package manager use the command (assuming version 0.1.0 of the library)

Install-Package Profanity.Detector -Version 0.1.0

Or via the command line

dotnet add package Profanity.Detector --version 0.1.0

# Example Usage

The following are some example of the basic usage of the library. You first need to either download or clone the code from this repository and include it in your project, or include the nuget package [Profanity.Detector](https://www.nuget.org/packages/Profanity.Detector/)

## Check if a word is classed as a profanity

The simplest scenario is to check if a word exists in the profanity list. This is done with a call to IsProfanity, and this performs a case insitive lookup in the profanity list.

```csharp
// Return true is a bad word
var filter = new ProfanityFilter();
Assert.IsTrue(filter.IsProfanity("arsehole"));

// Return false if NOT a naughty word
var filter = new ProfanityFilter();
Assert.IsFalse(filter.IsProfanity("fluffy"));
```

## Return list of all profanities in a sentence

The second scenario is returning a list of profanities from supplied string. This method will attempt remove any false positives from the string. For example, to quote the standard scunthope problem without removing the false positive, the word scunthorpe would report the word "c*nt" as this is contained within the word scunthorpe. This library will detect if a profanity is inside another word and filter it out if the enclosed word is also not a profanity.

```csharp
var filter = new ProfanityFilter();
var swearList = filter.DetectAllProfanities("2 girls 1 cup is my favourite twatting video");
Assert.AreEqual(3, swearList.Count);
Assert.AreEqual("2 girls 1 cup", swearList[0]);
Assert.AreEqual("twatting", swearList[1]);
```

## Censoring a Sentence

The third scenario is to provide a string containing text tha potentialy has profane language, and censor the text by replacing naughty words with a designated character like an '*'.

```csharp
var filter = new ProfanityFilter();

var censored = filter.CensorString("Mary had a little shit lamb who was a little fucker.");
var result = "Mary had a little **** lamb who was a little ******.";

Assert.AreEqual(censored, result);
```

# The Scunthorpe Problem

A common problem with profanity detector is solving what is called the [Scunthorpe Problem](https://en.wikipedia.org/wiki/Scunthorpe_problem). This is where you get a false positive result from a profanity detector because a profanity pattern is found inside a non profane word. For example, with "scunthorpe" (which is a town in the United Kingdom), it will get reported as containing the word "c*nt". What this proganity detector library will do is allow you to guard against this problem in two ways. The first is by using a whitelist of words that are to be excluded from the profanity detector. This is covered in the next section.

The second solution is to be a bit more inteligent about how we check in the string. What this library will do, in the scunthope example, is it will first detect the word "c*nt" in the string. Then the library will seek backwards and forward in the string to identify if that profanity is enclosed within another word. If it is, that enclosed word is checked against the profanity list. If that word is not in the list, which scunthorpe isn't, then the word is ignored. If that enclosed word is in the profanity list, then it will be reported as so. 

# Whitelisting

If there is a word in the profanity list that you don't consider a profanity, and you want to allow it through, you can add that word to a whitelist, so if that word appears in the input string, it will be ignored. In the example below we have the sentence, "You are a complete twat and a total tit."). In this example we want to say that the word "tit" is acceptable, so it gets added to the whitelist, this means the only reported profanity for that sentence is the word "twat".

```csharp
var filter = new ProfanityFilter();
filter.WhiteList.Add("tit");

var swearList = filter.DetectAllProfanities("You are a complete twat and a total tit.", true);

Assert.AreEqual(1, swearList.Count);
Assert.AreEqual("twat", swearList[0]);  
```

# Adding and Removing Profanties

There are a huge amount of words in the default list. The default list was put together from multiple lists online, so I, the author of this library, didn't physically write the list. If you feel that a word or words in the list are not what you consider to be a profanity, you can remove them via code, like in the following example. In the example we first check that "shit" is a profanity, and this returns true. Then we remove "shit" from the list and check if it is a profanity again. This time it returns true as we have removed it. 

```csharp
var filter = new ProfanityFilter();

Assert.IsTrue(filter.IsProfanity("shit"));
filter.RemoveProfanity("shit");

Assert.IsFalse(filter.IsProfanity("shit"));  
```

There may also be an occation where there is a word you want to include to the list that is not on the detault list. This can be easily done as in the following example. In this example we have deemed the word "fluffy" to be a profanity. We first check if it is a profanity, which returns false. Then we add "fluffy" to the list of profanities and check again which will return true.

```csharp
var filter = new ProfanityFilter();
Assert.IsFalse(filter.IsProfanity("fluffy"));

filter.AddProfanity("fluffy");
Assert.IsTrue(filter.IsProfanity("fluffy")); 
```

You can also add an array of words to the list aswell if you want to add them in one go. This is demonstrated by the following example. Here we are adding three new words to the list as an array.

```csharp
string[] _wordList =
{
"wibble",
"bibble",
"bobble"
};

var filter = new ProfanityFilter();
filter.AddProfanity(_wordList);
```

You can also directly add a List<string> instead of an array.
  
```csharp
string[] _wordList =
{
  "wibble",
  "bibble",
  "bobble"
};

var filter = new ProfanityFilter();
filter.AddProfanity(new List<string>(_wordList));
```

# Replacing the Profanitiy List

While developing this library, I had many people reach out to me to say that their companies maintains a signed off and curated list of profanities that they have to check for and therefore can't use the default list build into this Profanity Detector. This is a great suggestion, so I have tweaked the libray to allow completly overriding the detault list and adding your own.

In this first example we pass in an array of words into the ProfanityFilter constructor. This will stop the default list from being loaded and only insert these three words. This now means the profanity filter only contains three words, wibble, bibble, and bobble.

```csharp
string[] _wordList =
{
  "wibble",
  "bibble",
  "bobble"
};

IProfanityFilter filter = new ProfanityFilter(_wordList);
Assert.AreEqual(3, filter.Count);
```

You can also insert the new word list as a List<string>.
  
```csharp
string[] _wordList =
{
  "wibble",
  "bibble",
  "bobble"
};

IProfanityFilter filter = new ProfanityFilter(new List<string>(_wordList));
Assert.AreEqual(3, filter.Count);
```

Another was you can do this is to construct the ProfanityFilter with the default constructor that loads the default list, but then manually clear the list and insert your own array or List<string>.

```csharp
string[] _wordList =
{
  "wibble",
  "bibble",
  "bobble"
};

IProfanityFilter filter = new ProfanityFilter();
filter.Clear();
Assert.AreEqual(3, filter.Count);
```

# Frequently Asked Questions

**(Q)** Why does word (x) appear in the list, I don't consider it a profanity?
**(A)** The default list is compiled from lists I found on the internet that are allegedly used by some social media companies. On my first inspection of the list I did remove some words that I thought were not profane (in my opinion). It is possible I have missed some as the list is *HUGE*. It could also be that what is profane to one person, is not to another. 

If you spot something that you want to challenge, raise an issue and I will take a look. In the meantime, if there is a word that you don't agree with being on the list, you can manually whitelist it, as demonstrated above, or insert your own list.

**(Q)** Why have a profanity filter in the first place? Freedom of speech should not include censorship.
**(A)** I also agree in freedom of speech and don't neccesarily like censorship, except content to children, or hate speech, but in a lot of organizations there are requirements to check for profanities in a users input. If you are working in this type of environment, and a lot of companies do this, then you have to implement it; which is why this library exists.

**(Q)** My company has their own signed off list of profanities that needs to be censored on our system. Therefore I can't use the default list. Can I use my own.
**(A)** Of course, many people asked for this, so you can insert your own array/list of profanities by passing them into the ProfanityFilter constructor. See the example easlier in this readme file.
