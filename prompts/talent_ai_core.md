/TalentAI
├── /extension
│   ├── manifest.json      # Extension configuration
│   ├── content.js        # Scrapes job portals (Step 2)
│   ├── popup.html        # Extension UI (Resume selection)
│   └── popup.js          # Logic for selecting resume & calling AI
├── /web-app
│   ├── index.html        # Main Dashboard (Step 1)
│   ├── style.css         # Styling for both Dashboard & Extension
│   └── app.js            # Resume analysis, checkbox logic, & PDF logic
├── /prompts
│   └── system_prompt.md  # The AI rules you generated earlier
└── README.md

{
  "manifest_version": 3,
  "name": "TalentAI - Resume Optimizer",
  "version": "1.0",
  "permissions": ["activeTab", "storage", "scripting"],
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["*://*.linkedin.com/*", "*://*.indeed.com/*"],
      "js": ["content.js"]
    }
  ],
  "host_permissions": ["*://*.linkedin.com/*"]
}

// Step 2: Automatically read JD from the current page
function scrapeJobDetails() {
  const jobTitle = document.querySelector('.job-details-jobs-unified-top-card__job-title')?.innerText;
  const description = document.querySelector('#job-details')?.innerText;
  const company = document.querySelector('.job-details-jobs-unified-top-card__company-name')?.innerText;

  return {
    title: jobTitle || "Unknown Position",
    company: company || "Unknown Company",
    jdText: description || "No description found",
    domain: "UI/UX Design" // Extracted via AI logic later
  };
}

// Listen for messages from the popup
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.type === "GET_JD") {
    const details = scrapeJobDetails();
    sendResponse(details);
  }
});

// TalentAI State Management
let uploadedResumes = [];
const MAX_RESUMES = 5;

// Step 1: Upload and Analyze
function handleFileUpload(file) {
    if (uploadedResumes.length >= MAX_RESUMES) {
        alert("Maximum 5 resumes allowed.");
        return;
    }
    
    const resumeObj = {
        id: Date.now(),
        name: file.name,
        content: "Raw PDF Data...", // Logic to be handled by pdf-lib
        atsScore: Math.floor(Math.random() * 40) + 50, // Placeholder
        status: "Original"
    };

    uploadedResumes.push(resumeObj);
    updateUI();
    runATSCheck(resumeObj);
}

// Step 1.7 - 1.9: Update Logic (Keep Both vs Replace)
function finalizeResumeUpdate(isReplace, oldId, newResumeData) {
    const updatedResume = {
        ...newResumeData,
        id: Date.now(),
        status: "Updated"
    };

    if (isReplace) {
        // Replace old with new
        uploadedResumes = uploadedResumes.map(r => r.id === oldId ? updatedResume : r);
    } else {
        // Keep both
        uploadedResumes.push(updatedResume);
    }
    
    showPopup(`Resume updated with ${updatedResume.atsScore}% ATS Score!`);
    updateUI();
}

// UI Hover Effects (Step 1.10 - 1.11)
function createResumeItemElement(resume) {
    const div = document.createElement('div');
    div.className = "resume-item";
    div.innerHTML = `
        <span>${resume.name}</span>
        <div class="actions">
            <i class="fas fa-eye icon-preview" title="Preview"></i>
            ${resume.status === "Updated" ? '<i class="fas fa-download icon-dl"></i>' : ''}
            <i class="fas fa-trash icon-delete"></i>
        </div>
    `;
    return div;
}

// Step 2.3: Process Processing Status
function showProcessing() {
    const analysisPane = document.querySelector('.analysis-pane');
    analysisPane.innerHTML = `<div class="loader">Comparing Resume with JD...</div>`;
}

# TalentAI - Adaptive Resume Intelligence

**TalentAI** is a high-fidelity resume optimization tool that ensures your layout remains 100% intact while improving your ATS visibility.

### 🚀 Key Workflows
1. **The Dashboard:** Upload up to 5 resumes. The AI analyzes missing skills (e.g., Figma for UI/UX) and offers a checkbox-based update system.
2. **The Extension:** Navigate to LinkedIn, open TalentAI, and watch the AI scrape the JD in real-time. It compares your selected resume to the specific job and suggests keywords.

### 🛠 Technical Constraints
- **Layout:** Uses coordinate-based text replacement (No generic re-renders).
- **Single Page Rule:** The system uses synonym compression to ensure 1-page resumes stay 1-page.
- **Privacy:** All processing is grounded in existing resume facts—no hallucinations.

### 📦 Installation
1. Clone the repo.
2. Load `/extension` into Chrome via `chrome://extensions` (Developer Mode).
3. Open `/web-app/index.html` to manage your master resumes.
