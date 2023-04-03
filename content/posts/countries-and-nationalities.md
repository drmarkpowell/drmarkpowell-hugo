---
title: "Countries and Nationalities"
date: 2023-03-12T18:22:18-07:00
draft: false
---
 
Here's another great puzzle to code a solution for.

[This week's puzzle from NPR](https://www.npr.org/2023/03/12/1162867188/sunday-puzzle-around-the-world-in-nine-words):

> This week's challenge is a spinoff of my on-air puzzle. Name two countries that have consonyms that are nationalities of other countries. In each case, the consonants in the name of the country are the same consonants in the same order as those in the nationality of another country. No extra consonants can appear in either name. The letter Y isn't used.

So if you remove all the vowels, the name of a country will equal the name of a nationality from a different country. Also, for our purposes a letter Y can be considered a vowel (ignored). Sounds good!

### Internet of Trivia Things

A convenient list of all countries can be found [here](https://copylists.com/geography/list-all-countries/)

...and a list of nationalities [here](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/664133/CH_Nationality_List_20171130_v1.csv)

### To the Swift!

Here's my solution:

```swift
    func doPuzzle() async throws {
        let countriesUrl = Bundle.main.url(forResource: "countries", withExtension: "txt")
        let countryNames = countriesUrl!.lines
        var countriesNoVowels: [String: String] = [:]
        for try await name in countryNames {
            let noVowels = name.lowercased()
                .filter({ notAVowel($0) })
                .map({ "\($0)" })
                .reduce("", +)
            countriesNoVowels[noVowels] = name
        }

        let nationalitiesUrl = Bundle.main.url(forResource: "nationalities", withExtension: "txt")
        let nationalityNames = nationalitiesUrl!.lines
        var nationalitiesNoVowels: [String: String] = [:]
        for try await name in nationalityNames {
            let noVowels = name.lowercased()
                .filter({ notAVowel($0) })
                .map({ "\($0)" })
                .reduce("", +)
            nationalitiesNoVowels[noVowels] = name
        }

        var countriesNoVowelsSet: Set<String> = []
        countriesNoVowels.keys.forEach {
            countriesNoVowelsSet.insert($0)
        }

        nationalitiesNoVowels.keys.forEach {
            if countriesNoVowelsSet.contains($0) {
                print("Match! \(countriesNoVowels[$0]!) \(nationalitiesNoVowels[$0]!)")
            }
        }
    }

    func notAVowel(_ character: Character) -> Bool {
        character != "a" &&
        character != "e" &&
        character != "i" &&
        character != "o" &&
        character != "u" &&
        character != "y"
    }
```

The beauty of Swift's string implementation lets us iterate over each character of the name of a country or a nationality with a function like **map** or **filter**. As we need to remove the vowels and keep only consonants, a call to **filter** handles that aspect of the work, leaving us with a character. Following that, a call to *map** to transform the character to string, and **reduce** to combine all of those single-character strings together, and we have our consonants-only names. Making these names the key values of a dictionary allows us to map them back to their original country or nationality name.

### The Need? for Speed

To keep things snappy, I also build a **Set<String>** of vowel-less country names so that I can call **.contains(_: String)** on that for each nationality to check for a match. As it [states in the Apple docs](https://developer.apple.com/documentation/swift/set/contains(_:)), when the elements are **Hashable**, the performance of a contains check is O(1), so it's extremely fast. Although I may be guilty of premature optimization for this small data set, let's just chalk this one up to sticking with best practices, shall we? ;)

### One out of Two Ain't Bad (Also Spoilers)

So unfortunately with these data sources, we have no association between country and nationality, so we can't automatically reject matches from the *same* country like Germany and German. However, if we allow our eyes to fall immediately to the end of the list of matches we see:

> Match! Lebanon Albanian

...which is great! This was indeed one of the accepted answers.

Now if you look ahead to next week's puzzle for this accepted answer, you can also see that there was a second solution that we didn't see!

> Ukraine --> Korean

Which is a shame because the puzzle stated that the solver needed to find both. The reason why our code didn't find this answer was because in our nationalities file we have entries for 

> North Korean
>
> South Korean

...but none for simply "Korean". That's politics for you, I suppose.

It is still a fun puzzle to code up a solution for though. So, until next week, happy solving!
