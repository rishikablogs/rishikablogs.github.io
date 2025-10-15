---
title: "Guess the Probability Distribution!"
date: 2025-10-08
description: "Test your intuition about probability distributions with this interactive quiz."
---

## Test Your Probability Intuition

Have you ever wondered how good your intuition is when it comes to ratios or probabilities? This interactive quiz challenges you to guess the probability distribution of various scenarios. You'll enter your estimated percentages for each outcome, and then see how close your guess was to the actual distribution.

I am not a frontend engineer, and have never really built anything beyond a simple webpage with basic HTML. However, given the age of powerful coding assistants, I was curious to see if it can help me build this interactive quiz. I used Roo with Gemini 2.5 Flash for this. Although it was able to help me build the first version pretty quickly, I spent a lot of time and tokens trying to get it to refine certain UI aspects, like udpating the slider values dynamically, displaying the correct answer on the sliders. I ultimately had to give up and try to understand the Javascript and CSS code it had produced, and tinker it myself. Surprisingly, it also had trouble following simple instructions sometimes, like when I asked it to increase the size of the sliders, it increased the font size of the answer instead :P. 
These struggles gave me a newfound appreciation for the hardwork and skill of frontend engineers. 

<style>
    body {
        font-family: Arial, sans-serif;
        margin: 20px;
        background-color: #f4f4f4;
        color: #333;
    }
    .quiz-container {
        background-color: #fff;
        padding: 25px;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        max-width: 600px;
        margin: auto;
    }
    h2 {
        color: #0056b3;
        margin-bottom: 20px;
    }
    .question {
        margin-bottom: 20px;
        font-size: 1.1em;
        font-weight: bold;
    }
    .option-item {
        display: flex;
        align-items: center;
        margin-bottom: 15px;
    }
    .option-item label {
        flex: 0 0 120px; /* Fixed width for labels */
        margin-right: 15px;
        font-weight: normal;
    }
    button {
        background-color: #007bff;
        color: white;
        padding: 10px 15px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-size: 1em;
        margin-top: 15px;
    }
    button:hover {
        background-color: #0056b3;
    }
    .score-display {
        margin-top: 20px;
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 5px;
        background-color: #e9ecef;
        display: none; /* Hidden by default */
    }
    .score-display p {
        margin: 5px 0;
    }
    .error-message {
        color: red;
        margin-top: 10px;
        display: none;
    }
    .quiz-question-block {
        background-color: #f9f9f9;
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 20px;
        margin-bottom: 25px;
        box-shadow: 0 1px 3px rgba(0,0,0,0.05);
    }
    .quiz-question-block h4 {
        color: #0056b3;
        margin-top: 0;
        margin-bottom: 15px;
    }
    .score-display {
        margin-top: 20px;
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 5px;
        background-color: #e9ecef;
        display: none; /* Hidden by default */
    }
</style>
<div class="quiz-container">
    <h2>Guess the Probability Distribution</h2>
    <div id="quiz-questions-container">
        <!-- All questions will be dynamically loaded here -->
    </div>
    <div id="final-score-display" class="score-display" style="display:none;">
        <h3>Overall Quiz Results</h3>
        <p>Average Score: <span id="average-score-value"></span></p>
    </div>
</div>

<script>
    let quizData = []; // Will store all quiz questions
    let questionScores = []; // Will store scores for each question

    document.addEventListener('DOMContentLoaded', () => {
        fetch('interactive_quizzes/quiz_data.json')
            .then(response => response.json())
            .then(data => {
                quizData = data;
                loadAllQuizQuestions();
            })
            .catch(error => {
                console.error('Error loading quiz data:', error);
                document.getElementById('quiz-questions-container').innerHTML = '<p>Error loading quiz. Please try again later.</p>';
            });
    });

    function loadAllQuizQuestions() {
        const quizQuestionsContainer = document.getElementById('quiz-questions-container');
        quizQuestionsContainer.innerHTML = ''; // Clear previous content

        quizData.forEach((questionData, qIndex) => {
            const questionDiv = document.createElement('div');
            questionDiv.classList.add('quiz-question-block');
            questionDiv.innerHTML = `
                <div class="question">
                    ${qIndex + 1}. ${questionData.question}
                </div>
                <div class="options" id="options-container-${qIndex}">
                    <!-- Options will be dynamically loaded here for each question -->
                </div>
                <div class="error-message" id="error-message-${qIndex}" style="display:none;"></div>
                <div class="error-message" id="lock-error-message-${qIndex}" style="display:none;"></div>
                <button onclick="submitGuess(${qIndex})">Submit Guess</button>
                <div class="score-display" id="score-display-${qIndex}" style="display:none;">
                    <h4>Results for Question ${qIndex + 1}</h4>
                    <p>Your Guess: <span id="your-guess-${qIndex}"></span></p>
                    <p>Correct Distribution: <span id="correct-distribution-${qIndex}"></span></p>
                    <p>Your Score (Higher is better): <span id="score-value-${qIndex}"></span></p>
                </div>
            `;
            quizQuestionsContainer.appendChild(questionDiv);

            const optionsContainer = document.getElementById(`options-container-${qIndex}`);
            questionData.options.forEach((option, i) => {
                const optionItem = document.createElement('div');
                optionItem.classList.add('option-item');
                optionItem.innerHTML = `
                    <label for="slider-${qIndex}-${i}">${option}:</label>
                    <div>
                        <input type="range" id="slider-${qIndex}-${i}" data-question-index="${qIndex}" data-option-index="${i}" min="0" max="100" step="1" value="0" data-locked="false" list="tickmarks-${qIndex}-${i}">
                        <span id="value-${qIndex}-${i}">0.00%</span>
                        <datalist id="tickmarks-${qIndex}-${i}"></datalist>
                    </div>
                `;
                optionsContainer.appendChild(optionItem);
            });

        });

        // Attach event listeners to all sliders after they are created
        const sliders = document.querySelectorAll('input[type="range"]');
        sliders.forEach(slider => {
            slider.oninput = function() {
                const qIndex = parseInt(this.dataset.questionIndex);
                const oIndex = parseInt(this.dataset.optionIndex);
                updateSliderValues(qIndex, oIndex, this); // Pass the changed slider element
            };
        });

        // Initial update for all sliders
        quizData.forEach((_, qIndex) => updateSliderValues(qIndex, -1));
    }

    function updateSliderValues(questionIndex, changedOptionIndex, changedSliderElement = null) {
        const slidersForQuestion = document.querySelectorAll(`input[type="range"][data-question-index="${questionIndex}"]`);
        const valuesForQuestion = document.querySelectorAll(`span[id^="value-${questionIndex}-"]`);
        const slidersArray = Array.from(slidersForQuestion);
        const lockErrorMessage = document.getElementById(`lock-error-message-${questionIndex}`);
        const generalErrorMessage = document.getElementById(`error-message-${questionIndex}`);
        lockErrorMessage.style.display = 'none';
        generalErrorMessage.style.display = 'none';

        if (slidersArray.length === 0) return;

        // --- Step 1: Handle initial load or a slider set to 100% ---
        if (changedOptionIndex === -1) { // Initial load
            const initialValue = 100 / slidersArray.length;
            slidersArray.forEach(slider => {
                slider.value = initialValue.toFixed(0);
                slider.dataset.locked = "false"; // Ensure all are unlocked initially
            });
        } else { // A slider was changed by the user
            let changedSlider = changedSliderElement;
            let changedValue = parseInt(changedSlider.value);

            if (changedValue === 100) {
                slidersArray.forEach(slider => {
                    if (slider !== changedSlider) {
                        slider.value = 0;
                        slider.dataset.locked = "false";
                    }
                });
                changedSlider.value = 100;
                changedSlider.dataset.locked = "true";
                // Update display and exit early as state is perfectly 100%
                slidersArray.forEach((slider, i) => {
                    valuesForQuestion[i].textContent = `${parseInt(slider.value)}%`;
                });
                return;
            }
        }

        // --- Step 2: Determine locked/unlocked state and sums AFTER potential 100% adjustment ---
        let lockedSliders = slidersArray.filter(slider => slider.dataset.locked === "true");
        let unlockedSliders = slidersArray.filter(slider => slider.dataset.locked === "false");
        let sumOfLocked = lockedSliders.reduce((acc, slider) => acc + parseInt(slider.value), 0);

        // --- Step 3: Handle "sum of locked is 100" error for unlocked sliders ---
        if (changedOptionIndex !== -1 && changedSliderElement) {
            let changedSlider = changedSliderElement;
            let sumOfOtherLocked = 0;
            if (changedSlider.dataset.locked === "false") { // If the changed slider was unlocked before this interaction
                sumOfOtherLocked = lockedSliders.filter(s => s !== changedSlider).reduce((acc, s) => acc + parseInt(s.value), 0);
            } else { // If the changed slider was already locked
                sumOfOtherLocked = sumOfLocked; // All locked sliders sum
            }

            if (sumOfOtherLocked === 100 && changedSlider.dataset.locked === "false") {
                lockErrorMessage.textContent = `The sum of previously set values is already 100%. You cannot change this slider.`;
                lockErrorMessage.style.display = 'block';
                changedSlider.value = 0; // Revert the slider value
                // Do not lock it, as the change was invalid.
                // Re-filter locked/unlocked lists after reverting changedSlider
                lockedSliders = slidersArray.filter(slider => slider.dataset.locked === "true");
                unlockedSliders = slidersArray.filter(slider => slider.dataset.locked === "false");
                sumOfLocked = lockedSliders.reduce((acc, slider) => acc + parseInt(slider.value), 0);
            } else {
                // If the change is valid, and it's a user interaction, lock the slider
                // This is where the slider gets locked if it wasn't 100% and not blocked by other locked sliders.
                if (changedSlider.dataset.locked === "false") { // Only lock if it wasn't already locked
                    changedSlider.dataset.locked = "true";
                    // Re-filter locked/unlocked lists after locking changedSlider
                    lockedSliders = slidersArray.filter(slider => slider.dataset.locked === "true");
                    unlockedSliders = slidersArray.filter(slider => slider.dataset.locked === "false");
                    sumOfLocked = lockedSliders.reduce((acc, slider) => acc + parseInt(slider.value), 0);
                }
            }
        }

        // --- Step 4: Distribute remaining percentage among unlocked sliders ---
        let remainingPercentage = 100 - sumOfLocked;
        let currentSumOfUnlockedValues = unlockedSliders.reduce((acc, slider) => acc + parseInt(slider.value), 0);

        if (unlockedSliders.length > 0) {
            if (currentSumOfUnlockedValues === 0) {
                let baseValue = Math.floor(remainingPercentage / unlockedSliders.length);
                let remainder = remainingPercentage % unlockedSliders.length;

                unlockedSliders.forEach((slider, index) => {
                    let value = baseValue;
                    if (index < remainder) {
                        value++;
                    }
                    slider.value = Math.max(0, Math.min(100, value));
                });
            } else {
                let distributedSum = 0;
                let newUnlockedValues = [];
                for (let i = 0; i < unlockedSliders.length; i++) {
                    let slider = unlockedSliders[i];
                    let proportion = parseInt(slider.value) / currentSumOfUnlockedValues;
                    let newValue = Math.round(remainingPercentage * proportion);
                    newUnlockedValues.push(Math.max(0, Math.min(100, newValue)));
                    distributedSum += newUnlockedValues[i];
                }

                let diff = remainingPercentage - distributedSum;
                if (diff !== 0) {
                    let lastUnlockedIndex = newUnlockedValues.length - 1;
                    if (lastUnlockedIndex >= 0) {
                        newUnlockedValues[lastUnlockedIndex] = Math.max(0, Math.min(100, newUnlockedValues[lastUnlockedIndex] + diff));
                    }
                }

                unlockedSliders.forEach((slider, index) => {
                    slider.value = newUnlockedValues[index];
                });
            }
        }

        // --- Step 5: Final check and display ---
        let finalSum = slidersArray.reduce((acc, slider) => acc + parseInt(slider.value), 0);
        if (finalSum !== 100) {
            generalErrorMessage.textContent = `Total sum is ${finalSum}%, but should be 100%. Please adjust the values.`;
            generalErrorMessage.style.display = 'block';
        }

        slidersArray.forEach((slider, i) => {
            valuesForQuestion[i].textContent = `${parseInt(slider.value)}%`;
        });
    }


    // Helper function for Kullback-Leibler Divergence
    function klDivergence(p, q) {
        let divergence = 0;
        for (let i = 0; i < p.length; i++) {
            const pi = p[i];
            const qi = q[i];

            if (pi === 0) {
                // If p_i is 0, the term p_i * log(p_i / q_i) is 0.
                continue;
            }
            if (qi === 0) {
                // If p_i > 0 and q_i = 0, KL divergence is infinite.
                return Infinity;
            }
            // Both pi and qi are guaranteed to be > 0 here
            divergence += pi * Math.log2(pi / qi);
        }
        return divergence;
    }

    // Jensen-Shannon Divergence
    function jensenShannonDivergence(p, q) {
        const m = p.map((pi, i) => (pi + q[i]) / 2);
        const jsd = (klDivergence(p, m) + klDivergence(q, m)) / 2;
        return jsd;
    }

    function calculateScore(guess, correct) {
        // Using Jensen-Shannon Divergence, transformed to be between 0 and 1 (higher is better)
        const jsd = jensenShannonDivergence(guess, correct);
        // Max JSD with log base 2 is 1. So, 1 - JSD will give a score from 0 to 1, where 1 is perfect.
        const score = 1 - jsd;
        return score.toFixed(4);
    }

    function submitGuess(qIndex) {
        const slidersForQuestion = document.querySelectorAll(`input[type="range"][data-question-index="${qIndex}"]`);
        let userGuessPercentages = [];
        let sum = 0;
        const errorMessage = document.getElementById(`error-message-${qIndex}`);
        errorMessage.style.display = 'none';

        slidersForQuestion.forEach(slider => {
            const value = parseFloat(slider.value);
            userGuessPercentages.push(value);
            sum += value;
        });

        if (Math.abs(sum - 100) > 0.01) {
            errorMessage.textContent = `Your probabilities for this question sum to ${sum.toFixed(0)}%. They must sum to 100%. Please adjust.`;
            errorMessage.style.display = 'block';
            return;
        }

        const userGuess = userGuessPercentages.map(p => p / 100);
        const questionData = quizData[qIndex];
        const score = calculateScore(userGuess, questionData.correctDistribution);
        questionScores[qIndex] = parseFloat(score); // Store the score for this question

        document.getElementById(`your-guess-${qIndex}`).textContent = userGuessPercentages.map(p => `${p.toFixed(0)}%`).join(', ');
        document.getElementById(`correct-distribution-${qIndex}`).textContent = questionData.correctDistribution.map(p => `${(p * 100).toFixed(0)}%`).join(', ');
        document.getElementById(`score-value-${qIndex}`).textContent = score;
        document.getElementById(`score-display-${qIndex}`).style.display = 'block';

        // Populate datalists with correct answer tickmarks after submission
        const correctDistribution = questionData.correctDistribution;
        correctDistribution.forEach((correctProb, i) => {
            const datalist = document.getElementById(`tickmarks-${qIndex}-${i}`);
            if (datalist) {
                datalist.innerHTML = '';
                const option = document.createElement('option');
                option.value = (correctProb * 100).toFixed(0);
                datalist.appendChild(option);
            }
        });

        // Check if all questions have been answered
        if (questionScores.filter(s => s !== undefined).length === quizData.length) {
            const totalScore = questionScores.reduce((acc, s) => acc + s, 0);
            const averageScore = totalScore / quizData.length;
            document.getElementById('average-score-value').textContent = averageScore.toFixed(4);
            document.getElementById('final-score-display').style.display = 'block';
        }
    }
</script>
