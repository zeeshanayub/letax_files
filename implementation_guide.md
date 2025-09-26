# Implementation Guide: D's Kitchen Intent Detection Node Updates

## Overview
This guide provides step-by-step instructions for implementing the updated Intent Detection Node prompt that starts with a neutral greeting and delays name collection until required.

## Implementation Steps

### 1. Update Initial Greeting Logic

**Replace the immediate name request with a neutral greeting:**

```javascript
// OLD CODE - Remove this
if (is_returning_customer && customer_name) {
  return "Welcome back, " + customer_name + ". How can I help today?";
} else {
  return "Can I take your name to get started?";
}

// NEW CODE - Use this instead  
if (is_returning_customer && customer_name) {
  return "Welcome back, " + customer_name + ". How can I help today?";
} else {
  return "Welcome to D's Kitchen. How can I help you today?";
}
```

### 2. Modify Intent Detection Flow

**Update the flow to listen for intent before collecting names:**

```javascript
// NEW: Listen for intent first
function handleInitialResponse(userResponse) {
  const intent = classifyIntent(userResponse);
  
  if (intent) {
    // Intent is clear, proceed with name collection if needed
    if (nameRequiredForIntent(intent) && !customer_name) {
      return getNamePromptForIntent(intent);
    } else {
      // No name needed or already have it
      return proceedWithIntent(intent);
    }
  } else {
    // Intent unclear, ask for clarification
    return "Just to confirm—are you looking to place a new order, ask a question about our menu, or something else?";
  }
}
```

### 3. Add Intent-Specific Name Collection

**Create context-appropriate name requests:**

```javascript
function getNamePromptForIntent(intent) {
  switch(intent) {
    case 'new_order':
      return "I'd be happy to help you place an order. Could I get your name for the order?";
    case 'change_order':
    case 'cancel_order':
      return "I can help you with your order. What name is the order under?";
    case 'general_info':
    case 'complaint_upset':
      // These typically don't require names upfront
      return null;
    default:
      return "Could I get your name to help you with that?";
  }
}

function nameRequiredForIntent(intent) {
  const nameRequiredIntents = ['new_order', 'change_order', 'cancel_order'];
  return nameRequiredIntents.includes(intent);
}
```

### 4. Handle Ordering Without Name

**Update the intercept logic for users who start ordering immediately:**

```javascript
function handleOrderingWithoutName(userResponse) {
  if (containsOrderItems(userResponse) && !customer_name) {
    return "Happy to help with that—could I quickly get your name for the order?";
  }
  return null;
}

function handleNameRefusal() {
  return "No problem, I can still help you with that.";
}
```

### 5. Update Transition Conditions

**Modify the node completion logic:**

```javascript
function canTransitionFromIntentDetection() {
  const intentIdentified = current_intent !== null;
  
  if (is_returning_customer) {
    return intentIdentified;
  } else {
    const nameHandled = customer_name || 
                       !nameRequiredForIntent(current_intent) || 
                       name_collection_refused;
    return intentIdentified && nameHandled;
  }
}
```

## Testing Scenarios

### Test Case 1: Information Request
- **Input**: "What are your hours?"
- **Expected**: Direct response with hours, no name required
- **Verify**: No name collection prompt

### Test Case 2: New Order
- **Input**: "I'd like to place an order"
- **Expected**: "I'd be happy to help you place an order. Could I get your name for the order?"
- **Verify**: Context-appropriate name request

### Test Case 3: Complaint
- **Input**: "I have a complaint about my order"  
- **Expected**: "I can help you with your order. What name is the order under?"
- **Verify**: Name requested only because needed to look up order

### Test Case 4: Returning Customer
- **Input**: User identified as returning with stored name
- **Expected**: "Welcome back, [Name]. How can I help today?"
- **Verify**: Personalized greeting, no redundant name collection

## Configuration Updates

### Update System Prompts
```yaml
intent_detection:
  initial_greeting: "Welcome to D's Kitchen. How can I help you today?"
  returning_greeting: "Welcome back, {{customer_name}}. How can I help today?"
  
  name_prompts:
    new_order: "I'd be happy to help you place an order. Could I get your name for the order?"
    order_lookup: "I can help you with your order. What name is the order under?"
    ordering_intercept: "Happy to help with that—could I quickly get your name for the order?"
    
  clarification:
    intent_unclear: "Just to confirm—are you looking to place a new order, ask a question about our menu, or something else?"
    name_explanation: "Just so I can note your order correctly."
    
  refusal_handling:
    name_refused: "No problem, I can still help you with that."
```

## Rollback Plan

If issues arise, you can quickly rollback by reverting to the original prompt:

```javascript
// ROLLBACK: Use original aggressive name collection
function rollbackGreeting() {
  if (is_returning_customer && customer_name) {
    return "Welcome back, " + customer_name + ". How can I help today?";
  } else {
    return "Can I take your name to get started?";
  }
}
```

## Monitoring and Metrics

Track these metrics to validate the improvements:
- **Call completion rates** (should increase for info requests)
- **Customer satisfaction scores** (should improve)  
- **Average call duration for info requests** (should decrease)
- **Name collection success rate for orders** (should remain high)
- **Call abandonment rate** (should decrease)

## Notes for QA

- Ensure the neutral greeting doesn't break existing integrations
- Test with various regional accents and speech patterns
- Verify name collection still works reliably for order intents
- Check that the system properly handles edge cases (unclear intent, name refusal)
- Validate that returning customers still get personalized experience