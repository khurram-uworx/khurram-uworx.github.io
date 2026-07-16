If your original relationship was:

* **X** = engineering team size
* **Y** = business value delivered
* **Z** = added AI cost (licenses, inference, infrastructure, AI engineering effort)

then the equation changes depending on what AI actually does.

### Scenario 1: AI increases productivity (most common goal)

Previously:

[
Y = f(X)
]

After AI:

[
Y' = X \times P_{AI} - Z
]

Where:

* (P_{AI}) = productivity multiplier from AI (e.g. 1.2, 1.5, 2.0)
* (Z) = AI cost expressed in the same unit as value

The AI investment is worthwhile if:

[
X \times (P_{AI}-1) > Z
]

Meaning the additional value created exceeds the AI cost.

---

### Scenario 2: Maintain the same value with fewer engineers

If the goal is to deliver the same (Y):

[
Y = X_{\text{new}} \times P_{AI}
]

Therefore,

[
X_{\text{new}} = \frac{X}{P_{AI}}
]

Example:

* 100 engineers
* AI improves productivity by 25% ((P_{AI}=1.25))

Equivalent capacity:

[
100 \rightarrow 80 \text{ engineers}
]

assuming AI cost is lower than the cost of those 20 engineers.

---

### Scenario 3: Full economic model

A more realistic equation is:

[
\text{Net Value} = Y(X,P_{AI}) - Z - C_X
]

Where:

* (Y) = value delivered
* (C_X) = engineering payroll
* (Z) = AI costs (licenses, inference, GPUs, support)

ROI becomes:

[
ROI = \frac{Y_{\text{AI}} - Y_{\text{before}}}{Z}
]

or, more completely,

[
ROI = \frac{\Delta Y - Z}{Z}
]

---

### Executive framing

A useful way to think about it is:

> **Value Delivered = Engineering Capacity × AI Productivity Multiplier − AI Operating Cost**

or

> **Net Business Value = (Engineers × Productivity with AI) − Engineering Cost − AI Cost**

This framing makes it clear that AI is not "free productivity." It becomes economically beneficial only when the incremental value it enables exceeds its ongoing cost.
