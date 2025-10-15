---
title: "Guess the Probability Distribution!"
date: 2025-10-08
description: "Test your intuition about probability distributions with this interactive quiz."
---

## Test Your Probability Intuition

Have you ever wondered how good your intuition is when it comes to ratios or probabilities? This interactive quiz challenges you to guess the probability distribution of various scenarios. You'll enter your estimated percentages for each outcome, and then see how close your guess was to the actual distribution.

I am not a frontend engineer, and have never really built anything beyond a simple webpage with basic HTML. However, given the age of powerful coding assistants, I was curious to see if it can help me build this interactive quiz. I used Roo with Gemini 2.5 Flash for this. Although it was able to help me build the first version pretty quickly, I spent a lot of time and tokens trying to get it to refine certain UI aspects, like udpating the slider values dynamically, displaying the correct answer on the sliders. I ultimately had to give up and try to understand the Javascript and CSS code it had produced, and tinker it myself. Surprisingly, it also had trouble following simple instructions sometimes, like when I asked it to increase the size of the sliders, it increased the font size of the answer instead :P. 
These struggles gave me a newfound appreciation for the hardwork and skill of frontend engineers. 

[Launch the Probability Distribution Quiz!](interactive_quizzes/probability_quiz.html)

---

**Note:** This quiz uses a simple scoring mechanism (Squared Euclidean Distance) to measure the difference between your guess and the correct distribution.