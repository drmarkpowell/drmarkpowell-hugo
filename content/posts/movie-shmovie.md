---
title: "Movie Shmovie"
date: 2022-01-09T20:22:36-07:00
draft: false
---
 
Will Shortz himself has often said that however you want to solve the puzzle is fair game. I couldn’t come up with the solution in a short span today running it only in my head, and I have lots of other things to worry over in my spare cycles this week. That said, I figure why not have fun coding in Swift and solving the puzzle of the week at the same time?

This week's puzzle as stated at [NPR](https://www.npr.org/2022/01/09/1071581311/sunday-puzzle-movie-shmovie):

>This week’s challenge: This week’s challenge comes for Joseph Young, who conducts the blog “Puzzleria!” Let A = 1, B = 2, C = 3, etc. Think of a five-letter word whose letters’ values add up to 51. Now take this word’s last two letters. Add their values. (For example A and C would total 4.) Change these two letters to the single letter of the alphabet that represents their total. (In this case, D.) The result will be a new word that is the opposite of the original. What words are these?

Some things we can quickly glean from the stated puzzle:

1. The answer is two words that are opposites, one five letters long and the other four.
2. The first three letters of each of the pair of words is the same, so “light” and “dark” would not work. That’ll help us see the correct answer when we find it, though.

Time to write some code to automatically compute those point totals! 

### Five letter words

First, we need to find ourselves a text file containing all the five letter words that we want to test.

The [free dictionary](https://www.thefreedictionary.com/5-letter-words.htm) website has a very convenient list of five letter words that are in the Scrabble dictionary and several others. That’ll do nicely: copy and paste into a text file and we’re ready to read it with actual code and do some processing.

### It's Swift time

After firing up Xcode, here is the quick-fire code that will read in each word from the file and total up its point value:

 
```swift
    func doPointTotal() async throws {
        let fiveUrl = Bundle.main.url(forResource: "five_letter_words", withExtension: "txt")
        let wordLines = fiveUrl!.lines
        for try await line in wordLines {
            if line.count != 5 {
                continue
            }
            var pointTotal:UInt8 = 0
            for char in line.lowercased() {
                pointTotal += char.asciiValue! - Character("a").asciiValue! + 1
            }
            if pointTotal == 51 {
                print(line)
            }
        }
    }
```

The **URL.lines** returns a nice **AsyncSequence** that we can iterate over using this for try await loop. This will actually interleave the I/O of reading the file with the process, so the content will page in on demand if the file is really large, avoiding the need to wait to read it all first before processing can begin (or crash if there’s not enough memory capacity to read it all!) Three cheers for async/await in Swift!

Also, how elegant is that **for try await** syntax? Eat your heart out and my dust, Python, most popular language in the world! 

Once we have each word as a String, it’s a simple matter of computing the value of each character and assign “a” as 1, “b” as 2, and so on. If the point total of the word is exactly 51, we print it out.

Now before we write more code than we really need, let’s run it and see what we get:

> avoid
>
> broke
>
> daily

Looks like it’s working great! I actually got 14 words from my dictionary file that meet the puzzle criteria, and that narrows it down nicely, (The other words are omitted from this post to avoid spoilers for my fellow puzzle solving fans.)

That’s it for this week! Happy coding and puzzle solving. 
