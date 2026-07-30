```python
# Assigning 'fig', 'ax' variables.
fig, ax = plt.subplots(2, 2)

# Defining custom 'xlim' and 'ylim' values.
custom_xlim = (0, 100)
custom_ylim = (-100, 100)

# Setting the values for all axes.
plt.setp(ax, xlim=custom_xlim, ylim=custom_ylim) # 이게 핵심
```
