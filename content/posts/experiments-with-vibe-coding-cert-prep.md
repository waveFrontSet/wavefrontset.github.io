---
title: "Experiments With Vibe Coding: The Cert Prep Project"
date: 2026-03-23T20:00:00+01:00
slug: experiments-with-vibe-coding-cert-prep
categories:
  - programming
tags:
  - haskell
  - claude
  - codex
---

I can't remember where I read this, but someone once wrote that there are four
phases of a mathematician accepting new concepts and ideas:

1. "That's wrong."
2. "Well, it might not be wrong, but the old way is better."
3. "Well, there are cases where the new way is better."
4. "I've always done it that (new) way."

Regarding vibe coding, I've run through phases 1 and 2. To be completely
honest, I took pride in writing good code and reviewing it — especially in
reviews, there's always something new to learn, both for the reviewer and
the reviewee.

{{< notice info >}}
In case you haven't come across the term: vibe coding means describing what you
want in natural language and letting an AI agent write the code for you. You
review the output, steer the direction, but you're not writing most of the code
yourself.
{{< /notice >}}

Vibe coding seemed to take that part of learning away from me — or so I thought.
As it turned out, you can also use vibe coding as a tool to get rid of
boilerplate and to learn about the domain in a more systematic way.

## Vibe Coding Experiment: A Cert Prep Tool

I've always wanted to build something more complex in Haskell. And ever
since I started studying for cloud certifications, I had built a small TUI
app in Python to sample practice questions from a question pool and simulate
a certification exam (yes, some people might call this procrastination).
It didn't have all the bells and whistles, but it worked well enough.
When I found out about [the brick
library](https://hackage.haskell.org/package/brick)
for building TUIs in Haskell, I thought it would be a good learning
opportunity to rebuild this tool in Haskell.

So I started by creating the project foundations with a simple `cabal init`,
wrote the necessary types (questions, answers, configuration), some utility
functions, and a few tests. I thought this setup would serve as a good
guardrail for Claude Code — if you throw the AI agent into a project with
good conventions, it's more likely to follow them.
Plus, Haskell's strict and elaborate type system makes it
harder to make mistakes because conditions, restrictions, and rules
are already encoded at the type level.
Contrast this with Python, where type hints are entirely optional
(even though they are a firm part of modern Python code and libraries
such as [pydantic](https://docs.pydantic.dev/latest/)).

## Liftoff with Claude Code

Then I started prompting Claude Code to write the TUI app.
In less than an evening, I had feature parity with my old Python app —
and more: I also had stratified question sampling,
so I could simulate the distribution of question types in the real exam.

Over the first few iterations, my skepticism (phases 1 and 2, remember?) faded
almost completely. The code was clean, test cases were solid, and I got to
learn about
[lenses](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/a-little-lens-starter-tutorial),
which proved to be an even bigger rabbit hole than the whole Monad machinery.

I quickly had a configuration registry to gather and choose
between different question pools. Refactorings and reviews were
equally fast.

## Claude Code vs. Codex: Trophy System

Because Codex had a free tier offer until March 2026, I decided to
pit Claude Code against Codex for the shiniest feature so far:
a trophy system for the preparation, with a small animation popping up
when you earn one, similar to what happens in video games.

Here's the prompt I gave to both:

```text
You are a well-versed Haskell programmer who breathes monads. You love the type system, you love abstraction and
  self-documenting code.
  Your task: Introduce a trophy system to the application. Preferably, it should be added as an extra phase between
  answering / reviewing that checks whether the condition for a trophy has been satisfied.
  Come up with a good type for a trophy and create a few instances. Trophies instances should be attached to configs
  (i.e., I can win a trophy "5 correct answers in a row" for each config that I have). Come up with funny names similar
  to trophies on PS.
  In the app, each trophy should have a nice unique pixel icon. If possible, when achieving a trophy, this pixel icon
  should be shown after the review phase, with a big trophy animation and the text "Trophy achieved: <trophy name>".
```

### Claude Code's implementation

Claude Code's implementation took around *16 minutes* (around 10 minutes for
planning, 6 minutes for coding) with Opus 4.6.

It added the data model and the trophy conditions to `src/Types.hs` which
looked as follows (excerpt):

```haskell
data TrophyId
    = FirstBlood
    | HatTrick
    | OnFire
    | SpeedDemon
    | FlawlessVictory
    | ScholarSupreme
    | Marathoner
    deriving (Show, Eq, Ord, Enum, Bounded, Generic)

instance FromJSON TrophyId
instance ToJSON TrophyId

type EarnedTrophies = Set TrophyId

data TrophyDef = TrophyDef
    { trophyDefId :: TrophyId
    , trophyName :: Text
    , trophyDesc :: Text
    , trophyIcon :: [Text]
    }
    deriving (Show, Eq)

-- | Check for trophies earned after submitting an answer.
checkAfterSubmit ::
    Bool -> Int -> Int -> EarnedTrophies -> [TrophyDef]
checkAfterSubmit wasCorrect newStreak questionSeconds alreadyEarned =
    concatMap
        (filter (not . (`Set.member` alreadyEarned) . trophyDefId))
        [ [firstBloodDef | wasCorrect]
        , [hatTrickDef | wasCorrect, newStreak >= 3]
        , [onFireDef | wasCorrect, newStreak >= 5]
        , [speedDemonDef | wasCorrect, questionSeconds < 5]
        ]

-- | Check for trophies earned at the end of an exam.
checkAtFinish :: Int -> Int -> EarnedTrophies -> [TrophyDef]
checkAtFinish finalScore totalQs alreadyEarned =
    concatMap
        (filter (not . (`Set.member` alreadyEarned) . trophyDefId))
        [ [flawlessVictoryDef | finalScore == totalQs, totalQs > 0]
        , [scholarSupremeDef | totalQs >= 10, percentage >= 90]
        , [marathonerDef | totalQs >= 20]
        ]
  where
    percentage :: Int
    percentage
        | totalQs == 0 = 0
        | otherwise = (finalScore * 100) `div` totalQs
```

Deciphering the trophy conditions took me a while. At first I thought
it was using guard clauses, but then I realized it's using list
comprehensions where the condition doesn't depend on the list elements.
For instance, `[firstBloodDef | wasCorrect]` is equivalent to
`if wasCorrect then [firstBloodDef] else []`. Undeniably clever,
but also fairly confusing — not to mention the extra work involved
in unwrapping the list of lists with `concatMap`.

### Codex's implementation

Codex was blazingly fast, clocking in at around 4 minutes with no dedicated
planning phase.

It decided to put the trophy conditions into `app/TUI/Event.hs`

```haskell
isUnlockedBy :: ExamCore -> Bool -> Int -> Trophy -> Bool
isUnlockedBy core answerIsCorrect responseSeconds trophy =
    case trophyCondition trophy of
        CorrectStreakAtLeast n -> core ^. correctStreak >= n
        TotalCorrectAtLeast n -> core ^. score >= n
        FastCorrectAtMost n -> answerIsCorrect && responseSeconds <= n
```

while keeping the data model in `src/Types.hs`:

```haskell
data TrophyCondition
    = CorrectStreakAtLeast Int
    | TotalCorrectAtLeast Int
    | FastCorrectAtMost Int
    deriving (Show, Eq, Generic)

data TrophyIcon
    = PixelRocket
    | PixelFire
    | PixelCrown
    | PixelBolt
    deriving (Show, Eq, Generic)

data Trophy = Trophy
    { trophyId :: Text
    , trophyName :: Text
    , trophyCondition :: TrophyCondition
    , trophyIcon :: TrophyIcon
    }
    deriving (Show, Eq, Generic)
```

While there's an argument for putting code right where it's called,
I would have preferred keeping trophy conditions close to the data model.
Otherwise, adding a new trophy means touching at least two files instead
of one. That said, Codex's approach is arguably easier to understand
and makes good use of the lens machinery — at the cost of tightly coupling
trophy conditions to the state model of the event logic.

## My verdict

I slightly preferred Claude Code's approach of keeping the trophy conditions
close to the data model. However, I strongly disliked the tricky list
comprehensions — which is why I came up with my own variant that integrates
trophy conditions directly into the data model.

I used `DataKinds` and `TypeFamilies` to divide trophies into ones
checked after each answer and ones checked at the end of the exam.

```haskell
data TrophyState = TrophyState
    { currentStreak :: Int
    , lastQuestionSeconds :: Int
    }
    deriving (Show, Eq)

data FinalStatistics = FinalStatistics
    { nCorrectQuestions :: Int
    , nQuestions :: Int
    }

data TrophyCheckTime = WhenReviewing | WhenFinishing

type family CondInput (t :: TrophyCheckTime) where
    CondInput 'WhenReviewing = TrophyState
    CondInput 'WhenFinishing = FinalStatistics

data TrophyData (t :: TrophyCheckTime) = TrophyData
    { trophyDef :: TrophyDef
    , trophyCond :: CondInput t -> Bool
    }

```

Then I defined conditions for each trophy separately, tied to the data model.
Here's the "First Blood" trophy as an example (earned after answering the
first question correctly):

```haskell
-- Trophy definitions
currentStreakGte :: Int -> TrophyState -> Bool
currentStreakGte n ts = currentStreak ts >= n

firstBlood :: TrophyData 'WhenReviewing
firstBlood = TrophyData{trophyDef = firstBloodDef, trophyCond = currentStreakGte 1}
```

The check functions then become simple `map`s of `trophyCond` over the list of
all trophies.

```haskell
-- | Check for trophies earned after submitting an answer.
checkAfterSubmit :: Bool -> TrophyState -> [TrophyDef]
checkAfterSubmit wasCorrect ts =
    map trophyDef $
        filter
            ((&&) wasCorrect . flip trophyCond ts)
            [ firstBlood
            , hatTrick
            , onFire
            , speedDemon
            ]

-- | Check for trophies earned at the end of an exam.
checkAtFinish :: FinalStatistics -> [TrophyDef]
checkAtFinish fs =
    map trophyDef $
        filter
            (`trophyCond` fs)
            [flawlessVictory, scholarSupreme, marathoner]
```

## Conclusion

My main takeaway is that getting up and running with Claude Code, even in
a language you're not entirely familiar with, is incredibly fast. More than that,
I felt like I could learn relevant Haskell concepts (like lenses) much quicker
than I would have on my own — or I might not have used them at all.

The trophy feature also showed me that extending existing software with vibe
coding, while certainly producing functional results, will at some point lead
to technical debt. The trophy conditions I showed above might seem like a small
nuisance, but note that I haven't shown you the event code for the trophy
system: instead of introducing a separate phase for evaluating trophies,
Claude Code integrated them directly into the answering and final phase of the
exam. I haven't gotten around to refactoring that code yet, but it's
definitely on my list.

So where does that leave me on the four-phase scale? I think I'm solidly in
phase 3. There are clearly cases where vibe coding is better — spinning up a
project, plowing through boilerplate, exploring unfamiliar libraries. But I
still want to understand and reshape the code that comes out, and I don't see
that changing anytime soon. Phase 4 will have to wait.

If you want to take a look at the full code, I've published it
on [GitHub](https://github.com/waveFrontSet/cert-prep).
