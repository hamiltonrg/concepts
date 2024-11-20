#idea

##Title: Code Tasks
Description: 

- Input
  - Input: Prompt
    - LLM Process
      - Get intent
      - Check if intent makes sense with prompt if is something is missed
      - What assumptions are made to develop intent
        - What context is assumed if it hasn't been provided in the prompt
      - Are any assumptions unlikely or could cause major impact to success
      - If there are any risky assumptions, ask user for clarification or if the process should run again.
  - Input: Prompt + intent + context
    - What programmatic steps need to be taken to acccomplish
    - What design decisions need to be made
      - For each design decision
        - Input:Prompt + intent + context + design decision
          - Whats the most likely to be desired
          - Whats the best option
        - Input: Desired + best
        - Output: Design decision
  - Output / What we have
    - Prompt + intent + context + ordered list of programmatic steps with design decisions
      - *May need to revise this and develop design decisions before programmatic steps
  - Need to validate each programmatic step matches prompt + intent + context

- For each programmatic step
  - develop codee section for step + intent
  - section output = multiline comment
- for each section
  - Function definitions for step + section + intent
- for each function definition
  - define tests
for each function definition
  - write code in alignment with section comment + function definition
    - maybe write function comment first
  - ensure logging and debugging
    - debugging should be able to be dynamically turned on|off before and at runtime
- Execute tests
  - *Earliest develop tests should also have had a step to ensure they cover common tests and significant corner cases
  