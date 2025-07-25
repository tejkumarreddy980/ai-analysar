<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Photo Calorie Finder</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Google Fonts: Inter -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">

    <!-- Web App Manifest for PWA capabilities -->
    <link rel="manifest" href="data:application/json;base64,ewogICJuYW1lIjogIlBob3RvIENhbG9yaWUgRmluZGVyIiwKICAic2hvcnRfbmFtZSI6ICJQaG90b0NhbCIsCiAgImljb25zIjogWwogICAgewogICAgICAic3JjIjogImh0dHBzOi8vcGxhY2Vob2xkLmNvLzE5MngxOTIvM0I4M0ZDL0ZGRkZGRj90ZXh0PVBob3RvIiwKICAgICAgInR5cGUiOiAiaW1hZ2UvcG5nIiwKICAgICAgInNpemVzIjogIjE5MngxOTIiCiAgICB9LAogICAgewogICAgICAic3JjIjogImh0dHBzOi8vcGxhY2Vob2xkLmNvLzUxMng1MTIvM0I4M0ZDL0ZGRkZGRj90ZXh0PVBob3RvIiwKICAgICAgInR5cGUiOiAiaW1hZ2UvcG5nIiwKICAgICAgInNpemVzIjogIjUxMng1MTIiCiAgICAgfQogIF0sCiAgInN0YXJ0X3VybCI6ICIuIiwKICAiZGlzcGxheSI6ICJzdGFuZGFsb25lIiwKICAidGhlbWVfY29sb3IiOiAiIzFGMkE0NiIsCiAgImJhY2tncm91bmRfY29sb3IiOiAiI0Y5RkFGQiIKfQ==">

    <style>
        /* Custom styles for a better mobile experience */
        body {
            font-family: 'Inter', sans-serif;
            -webkit-tap-highlight-color: transparent; /* Removes blue tap highlight on mobile */
        }
        /* Simple spinner animation */
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #4f46e5; /* indigo-600 */
            animation: spin 1s ease infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        /* Fade-in animation for results */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
    </style>
</head>
<body class="bg-gray-50 dark:bg-gray-900 text-gray-800 dark:text-gray-200">

    <div class="container mx-auto p-4 max-w-lg">
        
        <!-- Header -->
        <header class="text-center my-8">
            <h1 class="text-4xl font-bold text-indigo-600 dark:text-indigo-400">Photo Calorie Finder</h1>
            <p class="text-gray-600 dark:text-gray-400 mt-2">Upload a photo to find the calories in your food.</p>
        </header>

        <!-- Image Upload Section -->
        <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-6 text-center">
            <input type="file" id="image-input" class="hidden" accept="image/*">
            <label for="image-input" class="cursor-pointer inline-block bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-8 rounded-full transition-colors duration-300">
                Select a Photo
            </label>
            <p class="text-xs text-gray-500 dark:text-gray-400 mt-3">Or drop an image file here</p>
        </div>
        
        <!-- Image Preview -->
        <div id="image-preview-container" class="mt-6 hidden">
             <img id="image-preview" src="#" alt="Image preview" class="max-w-full mx-auto rounded-lg shadow-md"/>
        </div>

        <!-- Results Section -->
        <main id="results-container" class="mt-6">
            <!-- Loading spinner will be injected here -->
            <!-- Results will be injected here -->
            <div id="initial-message" class="text-center text-gray-500 dark:text-gray-400">
                <p>Your food's calorie count will appear here.</p>
            </div>
        </main>
        
        <!-- "Add to Home Screen" Instructions for iOS -->
        <div id="ios-install-prompt" class="hidden fixed bottom-0 left-0 right-0 bg-gray-800 text-white p-4 text-center text-sm">
            To install, tap the 'Share' button and then 'Add to Home Screen'.
            <button id="close-ios-prompt" class="absolute top-2 right-2 text-xl">&times;</button>
        </div>

    </div>

    <script>
        // --- Application Elements ---
        const imageInput = document.getElementById('image-input');
        const imagePreviewContainer = document.getElementById('image-preview-container');
        const imagePreview = document.getElementById('image-preview');
        const resultsContainer = document.getElementById('results-container');
        const initialMessage = document.getElementById('initial-message');

        // --- Event Listener for Image Selection ---
        imageInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (file) {
                handleImage(file);
            }
        });

        /**
         * Handles the selected image file, displays a preview, and starts analysis.
         * @param {File} file - The image file selected by the user.
         */
        function handleImage(file) {
            const reader = new FileReader();
            reader.onload = (e) => {
                // Show image preview
                imagePreview.src = e.target.result;
                imagePreviewContainer.classList.remove('hidden');
                
                // Show loading spinner and start calorie fetching
                if(initialMessage) initialMessage.style.display = 'none';
                resultsContainer.innerHTML = `
                    <div class="flex justify-center items-center mt-8">
                        <div class="spinner"></div>
                    </div>
                `;
                
                // The result from FileReader is a Base64 string. We need to strip the metadata.
                const base64ImageData = e.target.result.split(',')[1];
                fetchCaloriesFromImage(base64ImageData);
            };
            reader.readAsDataURL(file);
        }

        /**
         * Fetches calorie data from the Gemini API using an image.
         * @param {string} base64ImageData - The Base64 encoded image data.
         */
        async function fetchCaloriesFromImage(base64ImageData) {
            const prompt = Analyze the food item in this image. Provide an estimate of its calorie count. If there are multiple items, focus on the main dish. If it's not food, say so.;
            
            // Define the desired JSON structure for the response
            const schema = {
                type: "OBJECT",
                properties: {
                    "foodName": { "type": "STRING" },
                    "calories": { "type": "NUMBER" },
                    "servingSize": { "type": "STRING" },
                    "isFood": { "type": "BOOLEAN" }
                },
                required: ["foodName", "calories", "servingSize", "isFood"]
            };

            const payload = {
                contents: [{
                    role: "user",
                    parts: [
                        { text: prompt },
                        { inlineData: { mimeType: "image/jpeg", data: base64ImageData } }
                    ]
                }],
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: schema,
                }
            };
            
            const apiKey = ""; // API key is handled by the environment
            const apiUrl = https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey};

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(API Error: ${response.statusText});
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates[0].content.parts[0].text) {
                    const data = JSON.parse(result.candidates[0].content.parts[0].text);
                    if (!data.isFood) {
                         displayError("This doesn't look like food. Please try another photo.");
                    } else {
                         displayResult(data);
                    }
                } else {
                    throw new Error("Could not analyze the image. The response was empty or malformed.");
                }
            } catch (error) {
                console.error("Error fetching calories from image:", error);
                displayError("Sorry, we couldn't analyze the image. Please try again.");
            }
        }

        /**
         * Displays the calorie result in a card.
         * @param {object} data - The calorie data object.
         */
        function displayResult(data) {
            resultsContainer.innerHTML = ''; // Clear spinner
            const resultCard = document.createElement('div');
            resultCard.className = 'bg-white dark:bg-gray-800 rounded-xl shadow-md p-6 animate-fade-in';
            
            resultCard.innerHTML = `
                <p class="text-lg text-gray-600 dark:text-gray-400">${data.foodName}</p>
                <div class="flex items-baseline justify-center text-center my-4">
                    <span class="text-7xl font-bold text-indigo-600 dark:text-indigo-400">${Math.round(data.calories)}</span>
                    <span class="text-xl text-gray-500 dark:text-gray-400 ml-2">kcal</span>
                </div>
                <p class="text-center text-gray-500 dark:text-gray-400">Serving Size: ${data.servingSize}</p>
            `;
            resultsContainer.appendChild(resultCard);
        }

        /**
         * Displays an error message.
         * @param {string} message - The error message to display.
         */
        function displayError(message) {
            resultsContainer.innerHTML = ''; // Clear spinner
            const errorCard = document.createElement('div');
            errorCard.className = 'bg-red-100 dark:bg-red-900 border border-red-400 dark:border-red-600 text-red-700 dark:text-red-200 px-4 py-3 rounded-lg relative animate-fade-in';
            errorCard.setAttribute('role', 'alert');
            errorCard.innerHTML = <strong class="font-bold">Oops!</strong><span class="block sm:inline ml-2">${message}</span>;
            resultsContainer.appendChild(errorCard);
        }

        // --- PWA Service Worker and Install Prompt Logic ---
        
        if ('serviceWorker' in navigator) {
            const swContent = `
                const CACHE_NAME = 'photo-calorie-finder-cache-v1';
                const urlsToCache = ['/', 'https://cdn.tailwindcss.com', 'https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap'];
                self.addEventListener('install', e => e.waitUntil(caches.open(CACHE_NAME).then(c => c.addAll(urlsToCache))));
                self.addEventListener('fetch', e => e.respondWith(caches.match(e.request).then(r => r || fetch(e.request))));
            `;
            const swBlob = new Blob([swContent], { type: 'application/javascript' });
            const swUrl = URL.createObjectURL(swBlob);
            window.addEventListener('load', () => {
                navigator.serviceWorker.register(swUrl).then(reg => console.log('SW registered')).catch(err => console.log('SW reg failed', err));
            });
        }

        const iosInstallPrompt = document.getElementById('ios-install-prompt');
        const closeIosPromptBtn = document.getElementById('close-ios-prompt');
        const isIOS = () => /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
        const isInStandaloneMode = () => ('standalone' in window.navigator) && (window.navigator.standalone);
        if (isIOS() && !isInStandaloneMode()) {
            iosInstallPrompt.classList.remove('hidden');
        }
        closeIosPromptBtn.addEventListener('click', () => {
            iosInstallPrompt.classList.add('hidden');
        });
    </script>
</body>
</html>
