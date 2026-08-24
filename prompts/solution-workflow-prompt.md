                 ┌─────────────────────┐
                 │ Master Prompt       │
                 │ solution-reference  │
                 └──────────┬──────────┘
                            │
                            ↓
              ┌─────────────────────────┐
              │ Single Question Prompt  │
              └────────────┬────────────┘
                           │
              Q1 → Solution
              Q2 → Solution
              Q3 → Solution
              ...
              Q65 → Solution
                           │
                           ↓
              ┌─────────────────────────┐
              │ Merge Exam Prompt       │
              └────────────┬────────────┘
                           │
                           ↓
                 ┌─────────────────────┐
                 │ solutions.md        │
                 │ Q1 → Q65            │
                 │ Pattern Recognition │
                 └──────────┬──────────┘
                            │
                            ↓
                       GitHub Repo