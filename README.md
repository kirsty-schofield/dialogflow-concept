# Conversation Flow Proof of Concept (Dialogflow)

A conversational AI agent built using Google Dialogflow designed to handle a user’s request to find out their account balance. 

## Objective & Use Case

In financial conversational design, the core challenge is balancing frictionless user experience with strict security and compliance guidelines. The assistant can’t simply reveal sensitive balance information without verifying the user’s identity, but forcing a user through a rigid, multi-step authentication form right at the start of a chat destroys the natural flow of conversation.


## Architecture & Flow Design

### Intent Architecture & Natural Language Mapping

To accurately parse user intent, I designed a multi-intent structure supported by System and Custom Entities.

Users approach conversational assistants with varying levels of specificity. To handle this, the model maps intent by identifying the user's primary goal (account.balance.check) alongside key parameters. If a user asks, "What's my balance?", the model triggers slot filling, prompting, "Which account would you like to check?".

### Custom Entity Strategy: account-type

Instead of expecting rigid terminology, I built a custom entity mapping reference values to common user synonyms:

-	Current Account: Current account, main account, everyday account.
-	Savings Account: Savings, ISA, nest egg, rainy day fund, emergency fund. 
-	Credit Card Account: Credit card, credit card account, VISA.

![Account Type image](https://github.com/kirsty-schofield/dialogflow-concept/blob/main/images/Account-type.png)

### Designing Contextual Logic Gates for Security

A critical design choice was never allowing a user to provide permanent sensitive credentials (like an ATM PIN or full account number) directly into a standard chat bubble over the internet. Doing so introduces severe security, logging, and compliance risks. Instead, I implemented a robust, state-managed authentication loop using Dialogflow ES Contexts as short-term memory logic gates.

The user identifies themselves by providing only the last 4 digits of their account number (mapped using the @sys.number-sequence system entity). This protects user privacy in chat histories.

Capturing the account digits triggers an output context named auth_in_progress with a strict lifespan of 2 turns.

To simulate a secure, production-grade One-Time Passcode (OTP) verification for this prototype without an active gateway, I mapped the identity.verify_otp intent to accept 6-digit numeric sequences. Once input, the model replaces auth_in_progress with an authenticated context pill.

The final account.balance.check intent is padlocked with an Input Context requirement of authenticated. If an unauthenticated user types "What's my balance?", the intent refuses to trigger, preventing unauthorised data exposure.


### Parameter Carryover 

Once a user successfully passes the authentication gate, they shouldn't have to restate what they came for. 

I configured the model to pass variables across the context chain. If a user starts the conversation by saying "Check my current account," the parameter account_type = checking is stored safely inside the auth_in_progress memory. By referencing the historical context string directly in the final response (#auth_in_progress.account_type), the bot dynamically outputs the balance. 


### Continuity & Error Handling 

A great conversational experience shouldn’t leave a user at a dead end. 
I rewrote the Default Fallback Intent to be context-aware. Instead of generic loop responses like "Say that one more time?", the fallback adapts to the authentication state, guiding users back to safety: "I'm having trouble verifying that information. If you're trying to check your balance, please provide your 4 digits, or type 'cancel' to start over."

To prevent an abrupt drop-off after delivering the balance, I appended a conversational hook: "Is there anything else I can help you with today?" and mapped quick-reply child intents (Yes / No) to smoothly transition the user into their next banking task or securely end the session.



## How to Use This Repo

Users can download the zip file to import into Google Dialogflow to view the training phrases, full system rules and fulfilment settings directly. 



