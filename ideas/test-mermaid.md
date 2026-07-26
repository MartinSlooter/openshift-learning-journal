## Examples
https://www.youtube.com/watch?v=JiQmpA474BY  
```mermaid
graph TD
  A-->B
```

```mermaid
flowchart
  S[Start] --> A
  A(Enter email) --> B{Existing user?}
  B -->|No| C(Enter name)
  C --> D{Accept conditions?}
  D -->|No| A
  D -->|Yes| E(Send email with link)
  B -->|Yes| E
  E --> End
```

## Export to pdf
https://mermaid.live/ can export to pdf.
