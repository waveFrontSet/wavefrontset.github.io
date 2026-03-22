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

I can't remember where I've read this, but someone wrote that there are four
phases for a mathematician to accept new concepts and ideas:

1. "That's wrong."
2. "Well, it might not be wrong, but the old way is better."
3. "Well, there are cases where the new way is better."
4. "I've always done it that (new) way."

Regarding vibe coding, I've run through phases 1 and 2; to be completely
honest, I took pride in writing good code and reviewing it: Especially in
reviews, there's always something new to learn (both for the reviewer and
the reviewee).

Vibe coding seems to take that part of learning away from me, or so I thought.
However, as it turned out, you can also use Vibe coding as a tool to get rid
of boilerplate code and to learn about the domain in a more systematic way.

## Vibe Coding Experiment: A Cert Prep Tool

Now, I've always wanted to build something more complex in Haskell. And ever
since I've started learning for Cloud certifications, I've built a small TUI
app in Python for myself in a couple of hours to sample practice questions from
a question pool to simulate a certification exam (yes, there are some people
who might call this procrastination).
It didn't really have all the bells and whistles but it worked well enough.
When I found out about [the brick
library](https://hackage.haskell.org/package/brick)
in Haskell that makes it possible to build TUIs in Haskell, I thought that it
would be a good learning opportunity to rebuild this certification prep tool
in Haskell.

And so I started to create the project foundations with a simple `cabal init`,
wrote the necessary types (questions, answers, configuration), some utility
functions and a few tests. I thought this setup would prove to be a good guard
rail for Claude Code (if I throw the AI agent into a project with good
conventions, it's more likely to follow them).
Plus, the strict and elaborate type system of Haskell makes it
harder to make mistakes because conditions, restrictions and rules
are already present on the type level.
Contrast this with Python, where type hints are entirely optional
(even though they are a firm part of modern Python code and libraries
such as [pydantic](https://docs.pydantic.dev/latest/)).

## Liftoff with Claude Code

And then I started to prompt Claude Code to write the TUI app.
In less than an evening, I had feature parity with my old Python app,
and more: I also had a feature for sampling questions in a stratified manner,
so that I could even simulate the distribution of question types in the real
exam.

For the first few iterations, my skepticism (phase 1 and 2, remember?) faded
almost completely. The code was clean, test cases were alright, and I could
even learn about
[lenses](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/a-little-lens-starter-tutorial)
which proved to be an ever bigger rabbit hole than the whole Monad machinery.

I quickly had a configuration registry so that I could gather and choose
between different question pools. Refactorings and reviews were
also quick.

## Claude Code vs. Codex: Trophy System

Because codex had a free tier offer till March 2026, I decided to
let Claude Code battle against Codex for the shiniest feature so far:
I wanted to integrate a Trophy System for the preparation with a small
animation popping up, similar to what happens in video games.

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

Deciphering the trophy conditions took me a while. First I thought that
it was using guard clauses, but then I realized that it is using list
comprehensions where the condition doesn't depend on the list elements!
For instance, `[firstBloodDef | wasCorrect]` is equivalent to
`if wasCorrect then [firstBloodDef] else []`. That's undeniably clever
but also fairly confusing, not to mention the little extra work involved
for unwrapping the list of lists with `concatMap`.

### Codex's implementation

Codex was blazingly fast, clocking in at around 4 minutes (no dedicated
planning phase).

Codex decided to put the trophy conditions into `app/TUI/Event.hs`

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
I would have favored putting trophy conditions close to the data model
of trophies. Otherwise, adding a new trophy would mean to touch at least
two files instead of one. Arguably, Codex's approach is easier to understand
and uses the Lens machinery - with the cost of tightly coupling trophy
conditions to the state model of the event logic.

## My verdict

I slightly preferred Claude Code's approach, keeping the trophy conditions
close to the data model. However, I strongly disliked the tricky list
comprehensions - which is why I came up with the following variant integrating
trophy conditions directly into the data model.

First, I used `DataKinds` and `TypeFamilies` to divide trophies into ones
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

Then, I defined the conditions for each trophy separately and tied
them to the data model. Here's an example for the first blood trophy (earned
after answering the first question correctly):

```haskell
-- Trophy definitions
currentStreakGte :: Int -> TrophyState -> Bool
currentStreakGte n ts = currentStreak ts >= n

firstBlood :: TrophyData 'WhenReviewing
firstBlood = TrophyData{trophyDef = firstBloodDef, trophyCond = currentStreakGte 1}
```

Then the check functions for all trophies become basic `map`s of `trophyCond` over the list of all
trophies.

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

The main takeaway for me is that getting up and running with Claude Code, even in
a language you're not entirely familiar with, is incredibly fast. Even more: I felt
like I could learn relevant Haskell concepts (like the Lens machinery) much quicker
-- or I might have not even used them at all.

The trophy feature also showed me that extending existing software with vibe
coding, while certainly leading to functional results, will at some point lead
to technical debt. In our case, the implementation of the trophy system might
seem to be a small nuisance, but note that I haven't actually shown you the
event code for the trophy system: Instead of introducing a separate phase
for evaluating trophies, Claude Code integrated them directly into the answering
and final phase of the exam. I haven't gotten to refactor that code yet, but
that is definitely something I'd like to do in the future.

In case you want to take a look at the code in its entirety, I've published it
at [GitHub](https://github.com/waveFrontSet/cert-prep).
