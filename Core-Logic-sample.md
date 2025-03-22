/**
 * FAQ Assistant - Core question matching and response flow
 * A sample implementation showing the key concepts
 */

// Sample FAQ database with structured entries
const faqDatabase = [
  {
    id: "setup-dev",
    question: "How do I set up the development environment?",
    answer: "To set up the development environment: \n" +
            "1) Clone the repo \n" +
            "2) Install dependencies \n" +
            "3) Configure environment variables \n" +
            "4) Run the dev server",
    keywords: ["setup", "environment", "development", "install"]
  },
  {
    id: "pr-workflow",
    question: "How do I submit a pull request?",
    answer: "To submit a PR: \n" +
            "1) Fork the repo \n" +
            "2) Clone your fork \n" +
            "3) Create a branch \n" +
            "4) Make changes \n" +
            "5) Push to your fork \n" +
            "6) Open a PR from the main repo",
    keywords: ["PR", "pull request", "contribute", "fork", "branch"]
  }
];

// System configuration options
const config = {
  matchThreshold: 0.85,
  actionType: "DM_WITH_APPROVAL", // Options: DIRECT_ANSWER, DM_WITH_APPROVAL, NOTIFY_ONLY
  notifyUsers: ["maintainer1", "maintainer2"]
};

/**
 * Main function to process an incoming question
 * 
 * @param {string} question - The user's question text
 * @param {Object} context - Contextual information about where/who asked
 * @returns {Object|null} - The answer object or null if no match found
 */
async function processQuestion(question, context) {
  console.log(`Processing question: "${question}"`);
  
  // Step 1: Match question to FAQ database
  const match = await matchQuestionToFAQ(question, faqDatabase, config.matchThreshold);
  
  if (!match.isMatch) {
    console.log("No matching FAQ found");
    return null;
  }
  
  console.log(`Matched FAQ: ${match.matchedFaqId} (confidence: ${match.confidence})`);
  
  // Step 2: Generate personalized answer for the user
  const matchedFAQ = faqDatabase.find(faq => faq.id === match.matchedFaqId);
  const answer = await generatePersonalizedAnswer(question, matchedFAQ, context);
  
  // Step 3: Take the appropriate action based on configuration
  await handleAction(config.actionType, answer, context, config);
  
  return answer;
}

/**
 * Match a question to the FAQ database
 * 
 * @param {string} question - The normalized user question
 * @param {Array} faqs - The FAQ database to search
 * @param {number} threshold - Minimum confidence threshold
 * @returns {Object} - Match results with confidence score
 */
async function matchQuestionToFAQ(question, faqs, threshold) {
  // Simplified matching logic for demonstration
  // Real implementation would use AI/LLM to understand question intent
  
  const normalizedQuestion = question.toLowerCase();
  let bestMatch = null;
  let bestScore = 0;
  
  for (const faq of faqs) {
    // Calculate a simple keyword-based match score
    const score = calculateMatchScore(normalizedQuestion, faq);
    
    if (score > bestScore && score >= threshold) {
      bestScore = score;
      bestMatch = faq.id;
    }
  }
  
  return {
    isMatch: bestMatch !== null,
    matchedFaqId: bestMatch,
    confidence: bestScore,
    reasoning: bestMatch 
      ? "Matched based on keywords and question similarity" 
      : "No good match found"
  };
}

/**
 * Calculate match score between question and FAQ entry
 * 
 * @param {string} question - Normalized user question
 * @param {Object} faq - FAQ entry to compare against
 * @returns {number} - Match score between 0-1
 */
function calculateMatchScore(question, faq) {
  let score = 0;
  
  // Check for keyword matches
  if (faq.keywords) {
    for (const keyword of faq.keywords) {
      if (question.includes(keyword.toLowerCase())) {
        score += 0.2;
      }
    }
  }
  
  // Check for overall question similarity
  // This is extremely simplified - real implementation would use embeddings or LLM
  if (question.includes(faq.question.toLowerCase().substring(0, 10))) {
    score += 0.5;
  }
  
  // Cap score at 1.0
  return Math.min(score, 1.0);
}

/**
 * Generate a personalized answer based on the matched FAQ
 * 
 * @param {string} question - Original user question
 * @param {Object} matchedFAQ - The matched FAQ entry
 * @param {Object} context - User and environment context
 * @returns {Object} - Formatted answer object
 */
async function generatePersonalizedAnswer(question, matchedFAQ, context) {
  // In a real implementation, this would use an LLM to generate a personalized response
  
  // Create personalized greeting and topic introduction
  const actionPhrase = matchedFAQ.question.toLowerCase().replace("how do i ", "");
  const personalization = `Hi @${context.questionerId}! Here's how you can ${actionPhrase}:`;
  
  // Combine personalized intro with answer content
  const answer = `${personalization}\n\n${matchedFAQ.answer}`;
  
  // Return structured answer object with metadata
  return {
    id: `answer-${Date.now()}`,
    originalQuestion: question,
    matchedFAQ: matchedFAQ.id,
    generatedAnswer: answer,
    context: context,
    timestamp: new Date().toISOString()
  };
}

/**
 * Handle the appropriate action based on configuration
 * 
 * @param {string} actionType - Type of action to take
 * @param {Object} answerData - The generated answer data
 * @param {Object} context - User and environment context
 * @param {Object} config - System configuration
 */
async function handleAction(actionType, answerData, context, config) {
  console.log(`Handling action: ${actionType}`);
  
  switch (actionType) {
    case "DIRECT_ANSWER":
      await postAnswerToSource(answerData, context);
      break;
      
    case "DM_WITH_APPROVAL":
      await sendApprovalRequests(config.notifyUsers, answerData);
      break;
      
    case "NOTIFY_ONLY":
      await sendNotifications(config.notifyUsers, answerData);
      break;
      
    default:
      console.log(`Unknown action type: ${actionType}`);
  }
}

/**
 * Example usage:
 * 
 * // Setup test context
 * const questionContext = {
 *   source: "github",
 *   repo: "owner/repo1",
 *   issueNumber: 123,
 *   questionerId: "user123"
 * };
 * 
 * // Process a question
 * processQuestion(
 *   "How do I set up the development environment for this project?", 
 *   questionContext
 * ).then(result => {
 *   console.log("Processing complete");
 * });
 */