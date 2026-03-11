---
name: Data Visualization
description: Generate Python scripts to create charts and data visualizations using matplotlib, executed via GitHub Actions
---

# Data Visualization Skill

**When to use:** "make a chart", "plot data", "visualize", "生成图表", "画图", "数据可视化"

## Rules
1. Generate self-contained Python scripts using matplotlib (pre-installed in GitHub Actions)
2. Save output as PNG/SVG files
3. Use clean, professional styling (no default matplotlib look)
4. Include proper labels, titles, and legends
5. Choose appropriate chart types for the data

## Chart Type Guide
| Data Type | Recommended Chart |
|-----------|------------------|
| Comparison | Bar chart (horizontal for many categories) |
| Trend over time | Line chart |
| Distribution | Histogram or box plot |
| Composition | Pie chart (≤6 slices) or stacked bar |
| Correlation | Scatter plot |
| Ranking | Horizontal bar chart |

## Output Format
```python
#!/usr/bin/env python3
"""Data visualization: [DESCRIPTION]"""
import matplotlib
matplotlib.use('Agg')  # Non-interactive backend for GitHub Actions
import matplotlib.pyplot as plt
import json

# Professional styling
plt.style.use('seaborn-v0_8-whitegrid')
plt.rcParams.update({
    'font.size': 12,
    'figure.figsize': (10, 6),
    'figure.dpi': 150,
})

# Data
data = {...}

# Create visualization
fig, ax = plt.subplots()
# ... plotting logic ...

ax.set_title('Title')
ax.set_xlabel('X Label')
ax.set_ylabel('Y Label')
plt.tight_layout()
plt.savefig('/tmp/chart.png', bbox_inches='tight')
print("Chart saved to /tmp/chart.png")
```

## With Email Delivery
When the user wants the chart emailed, encode the image as base64 and embed in HTML email:
```python
import base64
with open('/tmp/chart.png', 'rb') as f:
    img_b64 = base64.b64encode(f.read()).decode()
html = f'<img src="data:image/png;base64,{img_b64}" />'
```
