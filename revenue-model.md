# Revenue Model

```mermaid
graph LR
%% Colors %%
classDef blue fill:#66deff,stroke:#000,color:#000
classDef green fill:#6ad98b,stroke:#000,color:#000 
classDef red fill:#e8867d,stroke:#000,color:#000 
  
REVENUE --> SOFTWARE:::red
	REVENUE --> CONSULTING_HOURS:::red
	SOFTWARE --> PLATFORM:::blue
	SOFTWARE --> AI_INSIGHT:::blue
	SOFTWARE --> OTHER_ADDONS:::blue
	CONSULTING_HOURS --> SENIOR:::blue
	CONSULTING_HOURS --> MID_LEVEL:::blue
	CONSULTING_HOURS --> JUNIOR:::blue
```
