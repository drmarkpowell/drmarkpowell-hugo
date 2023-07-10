---
title: "Fill In The Blank"
date: 2023-07-09T18:22:18-07:00
draft: false
---
 
Here's another great puzzle to code a solution for.

[This week's puzzle from NPR](https://www.npr.org/2023/07/09/1186650754/sunday-puzzle-fill-in-the-blank):

> This week's challenge comes from listener Peter Gwinn, who writes for "Wait Wait ... Don't Tell Me!" Take the first name of a famous movie director. Write it in upper- and lowercase letters. Rotate the third letter of this name 180° and you'll get the name of the main character in one of this director's most popular movies. Who is it?

It's complicated to read, but it's easier than it sounds. Just turn the third letter of the first name upside-down and it should spell the name of a character the director is famous for in their films. Sounds good!

### IMDb Does Free Data

A convenient list of all actors, directors, etc. can be found [here](https://datasets.imdbws.com/name.basics.tsv.gz)

### To the Swift!

Here's my solution:

```swift
    func findAnswers() {
        let filePath = Bundle.main.path(forResource:"name.basics", ofType: "tsv")
        let contentData = FileManager.default.contents(atPath: filePath!)
        let content = String(data:contentData!, encoding:String.Encoding.utf8)!
        let substitutions = [ "b": "q", "q": "b", "p": "d", "d": "p", "n": "u", "u": "n"]
        for line in content.components(separatedBy: "\n") {
            let fields = line.components(separatedBy: "\t")
            if fields.count < 5 {
                continue
            }
            if fields[4].contains("director") {
                let name = fields[1]
                let firstName = name.components(separatedBy: " ").first ?? name
                if firstName.count < 4 {
                    continue
                }
                let thirdIndex = firstName.index(firstName.startIndex, offsetBy: 2)
                let third = String(firstName[thirdIndex])
                if substitutions.keys.contains(where: {$0 == third}) {
                    let fourthIndex = firstName.index(firstName.startIndex, offsetBy: 3)
                    let one = String(firstName[firstName.startIndex..<thirdIndex])
                    let two = substitutions[third]!
                    let three = String(firstName[fourthIndex..<firstName.endIndex])
                    let possibleAnswer = one + two + three
                    print("Character: \(possibleAnswer), Director Name: \(name)")
                }
            }
        }
    }
```

After dropping the tsv file into the project, I proceeded to load it in and parse it in a straightforward way first by lines and then separating out the tab-delimited fields. The person's name is the index 1 field and their best-known role list is in index 4. If they have a role that includes "director", we consider them.

There are only a subset of possible lowercase characters that legitimately look right when flipped upside-down. The possible substitutions I decided to look for included:

- u <-> n
- d <-> p
- b <-> q (Ok, I was feeling generous 😉)

### Swift Strings Could Be More Fun

Swift doesn't exactly make it easy to take pieces of strings out and reassemble them! Composing the required range indices into the string is a bit verbose. It was necessary to get 2 indices for me to code this: the first one to index on the third character, and the second one to index from the fourth character to end of the string to get the last portion of the name. Swift doesn't have a **>..** operator, or we could have expressed it as `[third>..firstName.endIndex]`. But, all we have to work with is `...` and `<..` operators, so another index is needed to get that last substring of the name.

With all the new possibilities that Swift 5.9 Macros allow for, I'm sure we won't have to wait too long for developers to make a nice package of String functionality enhancements that would make this sort of thing more concise and intuitive. It will be interesting to see what packages become part of the new mainstream in the near future...or even become subsumed into the Swift language for future official releases!