# Jobs MOC

```dataviewjs
// Collect all applications
let data = dv.pages('"jobs"');

// Define stages (excluding "None" since it's handled separately)
let stages = ["HR Interview", "First Interview", "Second Interview", "Culture Fit Interview"];

// Initialize counts
let totalApplications = data.length;

let counts = {
    total: totalApplications,
    directRejected: 0,
    waiting: 0, // Count for "Waiting" applications
    noneStage: 0, // Count for "None" stage
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

// Helper function to check if an application is waiting
function isWaiting(history) {
    return (
        !history || 
        history.length === 0 || 
        (history.length === 1 && (history[0] === "None" || history[0] === null))
    );
}

// Process applications
data.forEach(application => {
    let history = application.History;
    let status = application.Status;

    if (isWaiting(history)) {
        // Application is waiting
        if (status === "Rejected") {
            counts.directRejected++;
            // Transition from Total Applications to Rejected
            counts.transitions[`Total Applications,Rejected`] = (counts.transitions[`Total Applications,Rejected`] || 0) + 1;
        } else if (status === "Active") {
            counts.waiting++;
            counts.transitions[`Total Applications,Waiting`] = (counts.transitions[`Total Applications,Waiting`] || 0) + 1;
            counts.transitions[`Waiting,Active`] = (counts.transitions[`Waiting,Active`] || 0) + 1;
        }
    } else {
        // Has history and not waiting
        let lastStage = null;
        let rejected = false;

        for (let i = 0; i < history.length; i++) {
            let step = history[i];

            if (step === "None") {
                // Application is at 'None' stage
                counts.noneStage++;
                counts.transitions[`Total Applications,Waiting`] = (counts.transitions[`Total Applications,Waiting`] || 0) + 1;
                if (status === "Active") {
                    counts.transitions[`Waiting,Active`] = (counts.transitions[`Waiting,Active`] || 0) + 1;
                }
                break;
            } else if (stages.includes(step)) {
                counts.stageCounts[step]++;
                // Record transition from lastStage to current step
                if (lastStage) {
                    let key = `${lastStage},${step}`;
                    counts.transitions[key] = (counts.transitions[key] || 0) + 1;
                } else {
                    // Transition from Total Applications to first stage
                    let key = `Total Applications,${step}`;
                    counts.transitions[key] = (counts.transitions[key] || 0) + 1;
                }
                lastStage = step;
            } else if (step === "Reject") {
                // Rejected at lastStage
                if (lastStage) {
                    counts.rejectedAtStage[lastStage]++;
                    let key = `${lastStage},Rejected`;
                    counts.transitions[key] = (counts.transitions[key] || 0) + 1;
                } else {
                    counts.directRejected++;
                    counts.transitions[`Total Applications,Rejected`] = (counts.transitions[`Total Applications,Rejected`] || 0) + 1;
                }
                rejected = true;
                break; // Stop processing
            }
        }

        if (!rejected && status === "Rejected") {
            // Rejected without "Reject" in history
            if (lastStage) {
                counts.rejectedAtStage[lastStage]++;
                let key = `${lastStage},Rejected`;
                counts.transitions[key] = (counts.transitions[key] || 0) + 1;
            } else {
                counts.directRejected++;
                counts.transitions[`Total Applications,Rejected`] = (counts.transitions[`Total Applications,Rejected`] || 0) + 1;
            }
        } else if (!rejected && status === "Active") {
            // Application is still active at last stage
            if (lastStage) {
                counts.activeAtStage[lastStage]++;
                let key = `${lastStage},Active`;
                counts.transitions[key] = (counts.transitions[key] || 0) + 1;
            }
        }
    }
});

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
    // Ensure there are no commas or special characters in node names
    from = from.replace(/,/g, '');
    to = to.replace(/,/g, '');
    mermaidData += `${from},${to},${value}\n`;
}

mermaidData += "```";

// Render the chart
dv.span(mermaidData);
```


# Active
```dataview
table 
    Role as "Role", 
    Company as "Company", 
    Status as "Status", 
    row["Last Round"] as "Last Round",
    row["Date Applied"] as "Date Applied", 
    (date(today) - row["Date Last Interaction"]).days as "Days Since Last Interaction"
from "jobs"
where Status != "Rejected" and row["Date Applied"] >= date(2024-01-01)
sort row["Date Last Interaction"] desc
```
# Rejected
```dataview
table 
    Role as "Role", 
    Company as "Company", 
    Status as "Status", 
    row["Last Round"] as "Last Round",
    row["Date Applied"] as "Date Applied", 
    (date(today) - row["Date Last Interaction"]).days as "Days Since Last Interaction"
from "jobs"
where Status = "Rejected" and row["Date Applied"] >= date(2024-01-01)
sort row["Date Last Interaction"] desc
```

