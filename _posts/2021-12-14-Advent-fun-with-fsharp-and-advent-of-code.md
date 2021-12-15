---
layout: single
title: "Advent fun with F# and Advent of Code"
date: 2021-12-14 10:03:00
tags: ["F#", "F# Advent", "Advent of Code"]
slug: "advent-of-f#-code"
---
> This blog post is part of the [F# Advent 2021](https://sergeytihon.com/2021/10/18/f-advent-calendar-2021/), which brings you a new blog post every day, just as Advent of Code. Thank you, Sergey, for organising it again this year!

Every advent when the nights get longer and the days get colder (at least here in Europe), there is something to look forward to that even a pandemic can't take away. During advent, it is tradition to have a calendar which for each day provides a surprise. There are many variants, from those holding chocolate to beer or whatever may come to mind. There probably is a calendar out there providing just that. Since 2015 there is even an advent calendar providing daily coding challenges. [Advent of Code](https://adventofcode.com/) offers a coding puzzle from day 1 to 25 in December. The mystery does not dictate any language, and I really like to solve mine in F#. More on that later.

## The puzzle

A puzzle from Advent of Code always has the same structure. The puzzles are all embedded in a story related to Christmas. This year one is in a submarine looking for the keys of Santa sleigh that were accidentally dropped into the ocean. Each day a new piece of the story is revealed. And a new challenge is presented in the form of a puzzle that can be solved by programming.

The first one has to parse the input. There is a test input, which also has some sample answers in the text describing the issue. On day 14 (the day this blog post is intended to be released), we have to produce a polymer given a template input. Each letter pair of the information creates a new component in every step. The latest piece (a letter) is then inserted into the middle of the letter pair. You can read the full description of the puzzle [here](https://adventofcode.com/2021/day/14). A Puzzle provides a sample input and some sample solutions, which are great pointers when working on the algorithm to solve the daily quest. The riddle further comes in two parts. But part two will only reveal itself once the first puzzle is solved. The second part is a curveball. And can either change how the problem has to be solved or take it to another level.

## Probing the problem

Now why choose F# to solve such riddles? I really like to do it because F# allows me to start out using a script and the FSI REPL (which I already blogged about in a previous [blog post](https://mallibone.com/post/fsharp-scripting)). The FSI will enable me to quickly test out ideas and see how my solution progresses. For instance, every day, the input first has to be parsed. So I wrote the following helper functions, which reads in a given file.

```ocaml
let getTestInput day =
    let filename day = Path.Combine(__SOURCE_DIRECTORY__, $"Input/TestDay{day}.txt")
    File.ReadAllLines(filename day)

let getInput day =
    let filename day = Path.Combine(__SOURCE_DIRECTORY__, $"Input/Day{day}.txt")
    File.ReadAllLines(filename day)
```

So far, nothing special, but the following is to parse the input. The form of the information is as follows:

```
ABCD

AB -> C
BD -> A
...
```

So the first line is the starting template, followed by an empty line and then the instructions on how each pair produces a new element in the middle. E.g.: `AB -> ACB`

Parsing this is the following method we can add to our F# Script:

```ocaml
let parseInput (inputLines:string[]) =
    let initialInput = inputLines[0]

    let ruleSet =
        inputLines 
        |> Array.skip 2
        |> Array.map (fun l -> 
            let parts = l.Split(" -> ")
            parts[0], parts[1])
        |> Map.ofArray
    initialInput, ruleSet
```

At the end of the script we can now write the following lines of code to test this:

```ocaml
getTestInput 14
|> parseInput
```

Next, we will want to write the actual code. A naive implementation (the one I initially started out with) could look something like this:

```ocaml
let getNewElement (ruleset:Map<string,string>) (pairs:char[]) =
    let key = String.Join("", pairs)
    $"{pairs[0]}{ruleset[key]}"

let part1 input =
    let (template, ruleSet) = parseInput input
    let getNewElementFromRuleset = getNewElement ruleSet

    let rec runProcess processCount (processInput:string) =
        if processCount > 0 then
            processInput.ToCharArray() 
            |> Array.windowed 2
            |> Array.map getNewElementFromRuleset
            |> fun pairs -> $"""{String.Join("", pairs)}{template[template.Length - 1]}"""
            |> runProcess (processCount - 1)
        else
            processInput
		// ...

```

Here we use the windowing function of F# collections which allows us to really nicely create the pairs, then set the string back together. Then we have to sum up the occurrences and take the difference between the letter count that showed up the most and the least.

```ocaml
let part1 input =
		// ...
    runProcess 10 template
    |> (fun s -> s.ToCharArray())
    |> Array.groupBy id
    |> Array.map (fun (k, v) -> (k, v |> Array.length))
    |> Array.sortByDescending snd
    |> fun sorted -> (sorted |> Array.head |> snd) - (sorted |> Array.last |> snd)

getTestInput 14
|> part1
getInput 14
|> part1
```

So the FSI really lends itself to working on these problems. For one, it allows to try things out and get quick feedback. But it is also great for narrowing down bugs. 🙈 Plus, once FSI is up and running, it is pretty speedy and much faster than compiling code, running tests, and all. Therefore, lends itself better to fast iterative solutions, as is the case when solving these puzzles. The approach above us solves Part 1 of the puzzle. But you probably have guessed it by now.. part 2 is coming.

## The curveball

Part 2 is a twist, some catch or, in the case of day 14, the simple bump of the requirement to go from 10 to 40 steps to create a more robust polymer. The naive approach above just will not cut it at this point. Now coding puzzles are a field of development on their own. Some participants in the Advent of Code do coding challenges just like these all year round, and if you do that, you will see that many puzzles have a typical structure. And if you are new to this field or puzzles in general, it can be a great source to learn how to solve particular challenges you might encounter when writing code. There is an entire [subreddit](https://www.reddit.com/r/adventofcode/), and you will find many that solve the challenges while live coding and then uploading them to YouTube. I like to compare my solutions with [Jo van Eyck](https://github.com/jovaneyck/advent-of-code-2021), who has solved every Advent of Code challenge since the start of my Advent of Code journey.

I encourage you to check out the community and compare your approach with other solutions. Ideally, only read other answers once you have solved a day independently. But it can also be an excellent source for a hint to solve a puzzle. Especially when you have banged your head against a problem for too long. Now the question remains, how can we solve the above puzzle part 2? Let's look at how we can improve the performance of our solver. The key is not to grow the string but to use the pairs. So if we start out with our base template and window over it. We can then go over each step and produce the new couples, which we can sum together.

```ocaml
let getNextStep (ruleSet:Map<string,string>)(processInput:(string*int64)[]) =
    let nextInput = processInput 
                    |> Seq.collect (fun (k,v) -> 
                        // let chars = k.ToCharArray()
                        [|($"{k[0]}{ruleSet[k]}", v);($"{ruleSet[k]}{k[1]}", v)|])

                    |> Seq.groupBy fst
                    |> Seq.map (fun (k, v) -> (k, v |> Seq.sumBy snd))
                    |> Seq.toArray
    nextInput
```

So instead of creating an ever-larger growing string - and keep in mind, it grows pretty darn fast and large. We now only focus on the pairs and count how often they exist. We then can create the next step by using the count of the current pair and creating two new pairs based on the added letter. We now can invoke this step until we reach our step goal:

```ocaml
let part2 input =
    let (template, ruleSet) = parseInput input
    let getNewElementFromRuleset = getNewElement ruleSet

    let runProcessWithRuleSet = getNextStep ruleSet

    let pairs =
        template.ToCharArray() 
        |> Array.windowed 2
        |> Array.map(fun k -> String.Join("", k), 1L)

    let rec runProcess processCount (processInput:(string*int64)[]) =
        if processCount > 0 then
            let newProcessInput = 
                processInput
                |> runProcessWithRuleSet
            runProcess (processCount - 1) newProcessInput
        else
            processInput

    runProcess 40 pairs
    |> Array.map (fun (key, count) -> (key[0], count))
    |> Array.groupBy fst
    |> Array.map (fun (k, kv) -> (k, (kv |> Array.sumBy snd) + if k = template[(template.Length - 1)] then 1L else 0L))
    |> Array.sortByDescending snd
    |> fun sorted -> (snd sorted[0]) - (snd sorted[sorted.Length - 1])
```

In the end, we need to calculate the difference between the occurrences of the letter we see the most and the least.

I usually like to export the solution into a `.fs` file. For one, it is what I do in real-world apps where I first prototype in a `scratchpad.fsx` file and then export it to a "proper" F# file.



## Conclusion

Just like the F# Advent, the Advent of Code brings a new bit of joy every day during advent. And F# with all of it's powerfull list processing and .NET with it's general purpose functionality are a great fit to solve the daily challenges.

If you don't do so already I really hope you will try out advent of code and perhaps just like me learn some new tricks the language has to offer. Or dig out some old trigonometry Maths equations I haven't used in since forever.

You can find my journey of this years advent of code on [GitHub](https://github.com/mallibone/AdventOfCode2021) - and I am always happy to improve my F#. 😃

