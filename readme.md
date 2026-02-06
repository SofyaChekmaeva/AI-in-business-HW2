## Prompt 1.0

### Role  
You are an expert front-end web developer with deep knowledge of vanilla JavaScript, browser-based ML using Transformers.js, and static site deployment. You always follow the given specifications exactly and write clean, well-structured, production-ready code.

***

### Context  
We need a fully client-side web application that can be hosted as static files (for example on GitHub Pages or Hugging Face Spaces). The app should:

- Load a local TSV file named `reviews_test.tsv` containing a `text` column with product reviews.  
- Use Papa Parse in the browser to parse the TSV into an array of review texts.  
- On button click, select a random review, display it, and classify its sentiment using Transformers.js with a supported sentiment model such as `Xenova/distilbert-base-uncased-finetuned-sst-2-english`. [github](https://github.com/huggingface/transformers.js/blob/main/docs/source/tutorials/next.md)
- Run all inference directly in the browser (no Hugging Face Inference API calls) to avoid CORS issues and ensure the app remains purely static. [huggingface](https://huggingface.co/docs/transformers.js/index)

The UI should show:

- A text area containing the randomly selected review.  
- A sentiment/emotion result with a label (e.g., POSITIVE/NEGATIVE/NEUTRAL) and confidence score.  
- An icon indicating positive (thumbs up), negative (thumbs down), or neutral (question mark).  
- Loading and error states (e.g., while the model is loading or when TSV parsing fails).

***

### Instructions  

Implement the web app with the following exact requirements:

1. **File structure**  
   - Create exactly two files: `index.html` and `app.js`.  
   - `index.html` must contain the full HTML structure, inline CSS for basic styling, and script tags for external libraries.  
   - `app.js` must contain all JavaScript logic.  
   - Load `app.js` from `index.html` with a `<script type="module" src="app.js"></script>` tag so ES module imports are allowed.

2. **Libraries and CDNs**  
   In `index.html`, include:  
   - **Papa Parse** via CDN for TSV parsing:  
     - `<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>`  
   - **Font Awesome** for sentiment icons:  
     - `<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">`  
   - **Transformers.js** will be imported as an ES module from within `app.js` using:  
     - `import { pipeline } from "https://cdn.jsdelivr.net/npm/@huggingface/transformers@3.7.6/dist/transformers.min.js";` [huggingface](https://huggingface.co/docs/transformers.js/en/index)

3. **TSV loading and parsing (app.js)**  
   - Use `fetch("reviews_test.tsv")` to load the TSV file.  
   - Use Papa Parse with `header: true` and `delimiter: "\t"` to parse it. [papaparse](https://www.papaparse.com)
   - Extract an array of clean review texts from the `text` column, filtering out empty or non-string values.  
   - Handle network and parsing errors gracefully and show a user-friendly message in the UI.

4. **Model initialization with Transformers.js**  
   - In `app.js`, import `pipeline` from Transformers.js as an ES module.  
   - On startup (`DOMContentLoaded`), create a **single shared pipeline** for sentiment analysis or text classification:  
     - Example:  
       - `const sentimentPipeline = await pipeline("text-classification", "Xenova/distilbert-base-uncased-finetuned-sst-2-english");` [huggingface](https://huggingface.co/Xenova/distilbert-base-uncased-finetuned-sst-2-english)
   - Display a status message while the model is downloading/loading (e.g., “Loading sentiment model…”), and another when it is ready (e.g., “Sentiment model ready”).  
   - If model loading fails, log the error and show a readable error message in the UI.

5. **User interaction and sentiment analysis flow**  
   - Add a button labeled “Analyze random review”.  
   - When clicked:  
     - If no reviews are loaded, show an error message and do nothing else.  
     - Randomly select one review from the parsed list.  
     - Display the selected review in a dedicated element.  
     - Show a loading indicator and disable the button while analysis is running.  
     - Call the Transformers.js pipeline with the review text.  
       - The pipeline will return an array like `[{ label: "POSITIVE", score: 0.99 }, ...]`. [huggingface](https://huggingface.co/Xenova/distilbert-base-uncased-finetuned-sst-2-english)
     - Normalize this output into a structure that your display function can easily consume (e.g., choose the top result, or wrap as `[[{label, score}]]` if helpful).  
     - Map the `label` and `score` to one of three sentiment buckets: positive, negative, or neutral.  
       - Positive if label is `"POSITIVE"` and score > 0.5.  
       - Negative if label is `"NEGATIVE"` and score > 0.5.  
       - Neutral in all other cases.  
     - Update the UI with the label, confidence percentage, and the corresponding icon.  
     - Re-enable the button and hide the loading indicator when done.

6. **UI and styling details**  
   - The UI should include at minimum:  
     - A title for the page.  
     - A section showing the selected review text (use a `<div>` or `<p>`).  
     - A result area that shows:  
       - The sentiment label and confidence (e.g., “POSITIVE (98.7% confidence)”).  
       - A Font Awesome icon:  
         - Positive → `fa-thumbs-up`  
         - Negative → `fa-thumbs-down`  
         - Neutral → `fa-question-circle`  
     - A status text area for model loading messages.  
     - An error message area that is hidden by default and shown only when needed.  
   - Add simple, clear CSS to make the layout readable and visually distinct (e.g., different colors for positive/negative/neutral).

7. **Error handling**  
   - Handle and surface errors for:  
     - TSV load failures (network, 404).  
     - TSV parsing failures.  
     - Model loading failures.  
     - Inference failures (e.g., invalid output).  
   - Do not crash silently; log errors to the console and update the error message area with a user-friendly explanation.  
   - Always clear previous error messages when starting a new analysis.

8. **Technical constraints**  
   - Use only vanilla JavaScript; no frameworks or bundlers (no React, Vue, etc.).  
   - The app must run entirely in the browser with no server-side code or custom backend.  
   - Do not call the Hugging Face Inference API or Router; all inference must go through Transformers.js running locally in the browser. [github](https://github.com/huggingface/transformers.js)

9. **Code quality**  
   - Organize `app.js` into small, well-named functions for loading reviews, initializing the model, running analysis, displaying results, and handling errors.  
   - Use clear, concise comments in English explaining key logic, especially around model loading and sentiment mapping.  
   - Avoid global leaks other than the minimal necessary variables.

***

### Format  

Return your answer in this exact structure:

1. A complete `index.html` file in a fenced code block labeled `html`.  
   - Must include:  
     - Full HTML skeleton (`<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`).  
     - CSS styling (inline in a `<style>` tag is acceptable).  
     - Papa Parse and Font Awesome CDNs.  
     - The UI elements described above.  
     - `<script type="module" src="app.js"></script>` at the end of the body.  

2. A complete `app.js` file in a separate fenced code block labeled `javascript`.  
   - Must include:  
     - The `import { pipeline } from "https://cdn.jsdelivr.net/.../transformers.min.js";` statement.  
     - All logic for TSV loading, model initialization, random review selection, sentiment analysis with Transformers.js, UI updates, and error handling.  

3. Do not include any extra explanation or commentary outside these two code blocks.

## Prompt 2.0 (Logging)

### Project Context

I have a sentiment analyzer web application that uses Hugging Face Transformers.js to analyze product reviews locally in the browser. The application currently works but lacks logging functionality. I need to add Google Sheets logging to record each analysis result.

### Current Project Structure:

- `index.html` - Main HTML interface with review display and sentiment results
- `app.js` - Main JavaScript using Transformers.js for local sentiment analysis
- `reviews_test.tsv` - Sample reviews data file
- Deployed on GitHub Pages at: `https://github.com/dryjins/aib/tree/main/2026/l1/correct`
  
### Goal:

Add Google Sheets logging functionality that:
1. Logs each sentiment analysis result to Google Sheets
2. Includes timestamp, review text, sentiment with confidence, and metadata
3. Works with the existing UI and analysis flow

### Current Code State:

Here are the current files:

- `index.html (showing only relevant parts):`

html
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- Styles and meta tags -->
</head>
<body>
    <div class="container">
        <h3>Review Sentiment Analyzer</h3>
        
        <!-- Hugging Face API Token Section -->
        <div class="api-section">
            <h2>Hugging Face API Setup</h2>
            <div class="api-input">
                <label for="api-token">Enter your Hugging Face API Token:</label>
                <input type="text" id="api-token" placeholder="hf_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx">
            </div>
        </div>
        
        <!-- Analysis Section -->
        <div class="review-section">
            <button id="analyze-btn">Analyze Random Review</button>
            
            <div class="loading">Loading...</div>
            <div class="error" id="error-message"></div>
            
            <div class="review-card">
                <h3>Selected Review:</h3>
                <p class="review-text" id="review-text">Click the button above to analyze a random review</p>
            </div>
            
            <div class="sentiment-result" id="sentiment-result">
                <i class="fas fa-question-circle icon"></i>
                <span>Sentiment will appear here</span>
            </div>
        </div>
    </div>
    
    <script type="module" src="app.js"></script>
</body>
</html>

- `app.js (showing only main structure):`

javascript
import { pipeline } from "https://cdn.jsdelivr.net/npm/@huggingface/transformers@3.7.6/dist/transformers.min.js";

let reviews = [];
let sentimentPipeline = null;

document.addEventListener("DOMContentLoaded", function () {
    loadReviews();
    initSentimentModel();
    document.getElementById("analyze-btn").addEventListener("click", analyzeRandomReview);
});

function analyzeRandomReview() {
    // Gets random review, analyzes with sentimentPipeline, displays result
    // NEED: Add logging after successful analysis
}

// Other existing functions: initSentimentModel(), loadReviews(), displaySentiment(), etc.

### Requirements for Logging Feature
1. Google Apps Script Setup:
Create a Google Apps Script that:
- Receives data via GET requests (better for CORS)
- Writes to a Google Sheets with columns: ts_iso, Review, Sentiment (with confidence), Meta
- Handles test requests (?test=true) for connection testing
- Returns appropriate CORS headers
  
2. HTML Modifications Needed:
Add to index.html:
- A new section for Google Sheets configuration
- Input field for Google Apps Script URL
- Save and test connection buttons
- Visual status indicator for logging
  
3. JavaScript Modifications Needed:
Add to app.js:
- Functions to save/load Google Script URL from localStorage
- logSentimentAnalysis() function that sends data to Google Sheets
- Connection testing function
- Integration with existing analyzeRandomReview() function
- Error handling for logging failures (should not interrupt main analysis)
  
4. Data Format for Logging:
- Each log entry should include:
- ts_iso: ISO timestamp of analysis
- Review: The review text being analyzed (truncate if too long)
- Sentiment (with confidence): e.g., "POSITIVE (95.5% confidence)"
- Meta: JSON string containing metadata like:
  
json
{
  "model": "distilbert-base-uncased-finetuned-sst-2-english",
  "inferenceType": "local_transformers.js",
  "browser": "Chrome/120.0.0.0",
  "reviewLength": 142,
  "timestamp": "2024-01-15T10:30:00.000Z"
}

### Specific Implementation Instructions

#### Step 1: Modify index.html

Add this section after the existing API token section:

html
<!-- Google Sheets Logging Section -->
<div class="api-section">
    <h2><i class="fas fa-google"></i> Google Sheets Logging</h2>
    <div class="api-input">
        <label for="google-script-url">Google Apps Script URL:</label>
        <input type="text" id="google-script-url" 
               placeholder="https://script.google.com/macros/s/.../exec">
        <div style="display: flex; gap: 10px; margin-top: 10px;">
            <button id="save-script-url">Save URL</button>
            <button id="test-logging">Test Connection</button>
        </div>
        <p><small>Enter your Google Apps Script URL to enable logging of analysis results.</small></p>
    </div>
</div>

#### Step 2: Create Google Apps Script Code

Provide a complete Google Apps Script code that:
- Has doGet() function to handle GET requests
- Creates/accesses a sheet named "logs"
- Writes data in the specified format
- Handles test requests and errors gracefully

#### Step 3: Modify app.js

Add these specific functions:
- saveGoogleScriptUrl() - saves URL to localStorage
- loadGoogleScriptUrl() - loads URL on page load
- testGoogleSheetsConnection() - tests the connection
- logSentimentAnalysis(reviewText, sentimentLabel, confidence, meta) - sends data
- Update analyzeRandomReview() to call logging after successful analysis
  
#### Step 4: Integration Points

- The logging should happen AFTER sentiment analysis is complete and displayed
- Logging failures should NOT affect the main analysis flow (use try-catch)
- Show visual feedback when logging is successful/failed
- Store Google Script URL persistently in localStorage

### Constraints & Considerations

1. CORS Issues: Use mode: 'no-cors' with fetch or GET requests with URL parameters
2. Data Size: Truncate review text if > 10,000 characters for URL limits
3. Error Handling: Don't show logging errors to users unless they test connection
4. User Experience: Logging should be transparent to users (no extra clicks needed)
5. Performance: Logging should not slow down the analysis process
   
### Expected Final Behavior

1. User opens the page
2. Optionally configures Google Script URL (saved for future visits)
3. Clicks "Analyze Random Review"
4. Sentiment analysis happens and displays result
5. In background: data is sent to Google Sheets
6. User can verify by opening the Google Sheet
   
### Testing Requirements

1. Connection test should verify the Google Script is accessible
2. Each analysis should create a new row in Google Sheets
3. Logging should work even if user doesn't have Hugging Face API token
4. The system should handle network errors gracefully
   
### Output Format

Provide complete code for:
1. Modified index.html with logging UI
2. Modified app.js with all logging functions
3. Google Apps Script code for the backend
4. Brief setup instructions for Google Sheets and Apps Script

