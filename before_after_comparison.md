# Intent Detection Node - Before vs After Comparison

## Problem Summary
The original Intent Detection node was too aggressive in asking for caller names immediately, which felt forward and assumed all callers wanted to place orders. This created friction for information seekers, complainants, and others with non-order intents.

## Original Prompt (BEFORE)
```
Determine user intent efficiently and capture caller name when required.

Returning customer (is_returning_customer = true AND customer_name present): Say: "Welcome back, {{customer_name}}. How can I help today?"

New customer (is_returning_customer = false OR no name returned): First ask for their name before intent classification: "Can I take your name to get started?"

If they immediately start ordering without giving a name: politely intercept once: "Happy to help with that—could I quickly take your name first?"
If they ask why: "Just so I can note your order correctly."
If they refuse (info-only or reluctant): acknowledge and proceed anyway after one polite attempt.
Capture and store the name as customer_name (trim whitespace). If unclear name (e.g., noisy audio):

Clarify once: "I just want to be sure I heard that right—was it '[best guess]'?"
If still unclear, store a placeholder like "Guest" and continue (do NOT loop more than twice).
After greeting (and name capture if needed), classify intent into exactly one of:

new order
change order
cancel order
general info
complaint / upset
If the caller's intent is ambiguous, ask only ONE short clarifying question: "Just to confirm—are you looking to place a new order, ask a question, or something else?"

Only transition out of this node after:

Returning customer: greeting done + intent identified.
New customer: name captured (or refusal handled) + intent identified.
Do NOT start collecting menu items here. That happens after the Opening Hours Check.

Style: Natural, concise, one question at a time.
```

## Updated Prompt (AFTER)
```
Start with a neutral, welcoming greeting for all callers:
"Welcome to D's Kitchen. How can I help you today?"

Listen for caller's intent FIRST before requesting any personal information.

Returning customer (is_returning_customer = true AND customer_name present): 
Say: "Welcome back, {{customer_name}}. How can I help today?"

New customer - collect name ONLY when required for the specific intent:

For ORDER INTENTS (new order, change order, cancel order):
- New order: "I'd be happy to help you place an order. Could I get your name for the order?"
- Order changes/cancellation: "I can help you with your order. What name is the order under?"

For NON-ORDER INTENTS (general info, complaints):
- Proceed without requiring name initially
- Only ask for name if needed for specific resolution (e.g., looking up complaint)

If they start ordering without giving name: "Happy to help with that—could I quickly get your name for the order?"
If they ask why: "Just so I can note your order correctly."
If they refuse: "No problem, I can still help you with that." (proceed with "Guest")

Name clarification (if unclear audio):
"I just want to be sure I heard that right—was it '[best guess]'?"
Max 2 attempts, then use "Guest" placeholder.

Classify intent into exactly one of:
- new order
- change order
- cancel order
- general info
- complaint / upset

If intent ambiguous: "Just to confirm—are you looking to place a new order, ask a question about our menu, or something else?"

Transition conditions:
- Returning customer: greeting done + intent identified
- New customer: intent identified + (name captured when required OR name not required for intent)

Style: Natural, concise, one question at a time.
```

## Key Changes Made

### 1. **Neutral Opening Greeting**
- **Before**: Immediately asked new customers for name: "Can I take your name to get started?"
- **After**: Neutral greeting for all: "Welcome to D's Kitchen. How can I help you today?"

### 2. **Intent-First Approach** 
- **Before**: Name collection happened before understanding caller's needs
- **After**: Listen to caller's intent first, then collect name only if needed

### 3. **Conditional Name Collection**
- **Before**: Always asked for name from new customers regardless of intent
- **After**: Only request names when required for specific intents (orders, order changes)

### 4. **Improved Messaging**
- **Before**: Generic "Can I take your name to get started?" 
- **After**: Context-specific requests:
  - "Could I get your name for the order?" (new orders)
  - "What name is the order under?" (existing orders)

### 5. **Better Handling of Non-Order Intents**
- **Before**: Info seekers and complainants still had to provide names upfront
- **After**: Can proceed immediately with info requests without name friction

## Benefits Achieved

1. **More Welcoming**: Neutral greeting doesn't assume intent or create pressure
2. **Reduced Friction**: Info seekers can get answers immediately without name requirements  
3. **Natural Flow**: Conversation follows logical progression (intent → requirements)
4. **Maintained Efficiency**: Still captures necessary information when actually needed
5. **Better UX**: Supports wider range of use cases beyond just ordering

## Use Case Examples

### Information Request (Improved Experience)
- **Before**: "Can I take your name to get started?" → "I just want to know your hours"
- **After**: "Welcome to D's Kitchen. How can I help you today?" → "What are your hours?" → Direct answer

### Complaint (Improved Experience)  
- **Before**: "Can I take your name to get started?" → "I have a complaint" → Friction
- **After**: "Welcome to D's Kitchen. How can I help you today?" → "I have a complaint" → Natural response

### New Order (Same Efficiency)
- **Before**: "Can I take your name to get started?" → Name → "What can I get you?"
- **After**: "Welcome to D's Kitchen. How can I help you today?" → "I'd like to order" → "Could I get your name for the order?" → Name → Order