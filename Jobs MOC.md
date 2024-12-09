# Jobs MOC

This below is a more compact version of the graph. We define stages so to avoid cluttering the graph. Stages are defined with:

```javascript
let stages = ["Waiting", "HR Interview", "First Interview", "Second Interview", "Culture Fit Interview", "Offer"];
```

```dataviewjs
// Collect all applications
let data = dv.pages('"jobs"');

// Define stages, including "Waiting"
let stages = ["Waiting", "HR Interview", "First Interview", "Second Interview", "Culture Fit Interview", "Offer"];

// Initialize counts
let totalApplications = data.length;

let counts = {
    total: totalApplications,
    directRejected: 0,
    waiting: 0, // Count for "Waiting" applications
    stageCounts: {},
    rejectedAtStage: {},
    activeAtStage: {},
    transitions: {},
};

// Initialize counts for stages
stages.forEach(stage => {
    counts.stageCounts[stage] = 0;
    counts.rejectedAtStage[stage] = 0;
    counts.activeAtStage[stage] = 0;
});

// Helper functions
function initializeHistory(history) {
    if (!history || history.length === 0 || (history.length === 1 && (history[0] === "None" || history[0] === null))) {
        return ["Waiting"];
    }
    if (history[0] === "None" || history[0] === null) {
        history[0] = "Waiting";
    }
    return history;
}

function processRejection(counts, lastStage) {
    if (lastStage) {
        counts.rejectedAtStage[lastStage] = (counts.rejectedAtStage[lastStage] || 0) + 1;
        counts.transitions[`${lastStage},Rejected`] = (counts.transitions[`${lastStage},Rejected`] || 0) + 1;
    } else {
        counts.directRejected++;
        counts.transitions[`Total Applications,Rejected`] = (counts.transitions[`Total Applications,Rejected`] || 0) + 1;
    }
}

function processActive(counts, lastStage) {
    let stage = lastStage || "Waiting";
    counts.activeAtStage[stage] = (counts.activeAtStage[stage] || 0) + 1;
    counts.transitions[`${stage},Active`] = (counts.transitions[`${stage},Active`] || 0) + 1;
}

function recordTransition(transitions, from, to) {
    if (from !== to) {
        transitions[`${from},${to}`] = (transitions[`${from},${to}`] || 0) + 1;
    }
}

// Process applications
data.forEach(application => {
    let history = initializeHistory(application.History);
    let status = application.Status;

    let lastStage = null;
    let rejected = false;

    for (let step of history) {
        if (step === "Reject") {
            processRejection(counts, lastStage);
            rejected = true;
            break; // Stop processing further stages
        } else if (stages.includes(step)) {
            counts.stageCounts[step] = (counts.stageCounts[step] || 0) + 1;
            counts.activeAtStage[step] = counts.activeAtStage[step] || 0;
            counts.rejectedAtStage[step] = counts.rejectedAtStage[step] || 0;

            recordTransition(counts.transitions, lastStage || "Total Applications", step);
            lastStage = step;
        }
    }

    if (!rejected && status === "Rejected") {
        processRejection(counts, lastStage);
    } else if (!rejected && status === "Active") {
        processActive(counts, lastStage);
    }
});

// Set counts.waiting after processing
counts.waiting = counts.activeAtStage["Waiting"];

// Build Mermaid Sankey diagram
let mermaidData = "```mermaid\n";
mermaidData += "---\n";
mermaidData += "config:\n";
mermaidData += "  themeCSS: \"\"\n"; // Clears forced CSS, enabling base Mermaid styles
mermaidData += "---\n";
mermaidData += "sankey-beta\n";

// Generate transitions
for (let [key, value] of Object.entries(counts.transitions)) {
    let [from, to] = key.split(",");
    from = from.replace(/,/g, ''); // Ensure node names have no commas
    to = to.replace(/,/g, '');
    if (from === to) continue; // Skip self-transitions
    mermaidData += `${from},${to},${value}\n`;
}

mermaidData += "```";

// Render the chart
dv.span(mermaidData);
```


This is the full version of the graph but it can get a bit confusing.

```dataviewjs
// Collect all applications
let data = dv.pages('"jobs"');


// Initialize counts
let totalApplications = data.length;

let counts = {
    total: totalApplications,
    directRejected: 0,
    waiting: 0,
    stageCounts: {},
    rejectedAtStage: {},
    activeAtStage: {},
    transitions: {},
};

// Helper functions
function initializeHistory(history) {
    if (!history || history.length === 0 || (history.length === 1 && (history[0] === "None" || history[0] === null))) {
        return ["Waiting"];
    }
    if (history[0] === "None" || history[0] === null) {
        history[0] = "Waiting";
    }
    return history;
}

function processRejection(counts, lastStage) {
    if (lastStage) {
        counts.rejectedAtStage[lastStage] = (counts.rejectedAtStage[lastStage] || 0) + 1;
        counts.transitions[`${lastStage},Rejected`] = (counts.transitions[`${lastStage},Rejected`] || 0) + 1;
    } else {
        counts.directRejected++;
        counts.transitions[`Total Applications,Rejected`] = (counts.transitions[`Total Applications,Rejected`] || 0) + 1;
    }
}

function processActive(counts, lastStage) {
    let stage = lastStage || "Waiting";
    counts.activeAtStage[stage] = (counts.activeAtStage[stage] || 0) + 1;
    counts.transitions[`${stage},Active`] = (counts.transitions[`${stage},Active`] || 0) + 1;
}

function recordTransition(transitions, from, to) {
    if (from !== to) {
        transitions[`${from},${to}`] = (transitions[`${from},${to}`] || 0) + 1;
    }
}

// Process applications
data.forEach(application => {
    let history = initializeHistory(application.History);
    let status = application.Status;

    let lastStage = null;
    let rejected = false;

    for (let step of history) {
        if (step === "Reject") {
            processRejection(counts, lastStage);
            rejected = true;
            break; // Stop processing further stages
        } else {
            // Initialize stage counts if not exist
            counts.stageCounts[step] = (counts.stageCounts[step] || 0) + 1;
            counts.activeAtStage[step] = counts.activeAtStage[step] || 0;
            counts.rejectedAtStage[step] = counts.rejectedAtStage[step] || 0;

            // Record transition
            recordTransition(counts.transitions, lastStage || "Total Applications", step);
            lastStage = step;
        }
    }

    if (!rejected && status === "Rejected") {
        processRejection(counts, lastStage);
    } else if (!rejected && status === "Active") {
        processActive(counts, lastStage);
    }
});

// Set counts.waiting after processing
counts.waiting = counts.activeAtStage["Waiting"] || 0;

// Build Mermaid Sankey diagram
let mermaidData = "```mermaid\n";
mermaidData += "---\n";
mermaidData += "config:\n";
mermaidData += "  themeCSS: \"\"\n"; // Clears forced CSS, enabling base Mermaid styles
mermaidData += "---\n";
mermaidData += "sankey-beta\n";

// Generate transitions
for (let [key, value] of Object.entries(counts.transitions)) {
    let [from, to] = key.split(",");
    from = from.replace(/,/g, ''); // Ensure node names have no commas
    to = to.replace(/,/g, '');
    if (from === to) continue; // Skip self-transitions
    mermaidData += `${from},${to},${value}\n`;
}

mermaidData += "```";

// Render the chart
dv.span(mermaidData);
```

