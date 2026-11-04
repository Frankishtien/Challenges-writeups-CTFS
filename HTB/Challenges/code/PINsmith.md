# PINsmith


```python
from itertools import product

def solve(template):
    n = len(template)
    results = []
    
    # For each position, determine possible digits
    choices = []
    for ch in template:
        if ch == '*':
            choices.append('0123456789')
        else:
            choices.append(ch)
    
    # Iterate all combinations in lexicographic order
    for combo in product(*choices):
        valid = True
        for i in range(1, n):
            if combo[i] == combo[i-1]:
                valid = False
                break
        if valid:
            results.append(''.join(combo))
    
    # product already yields in lexicographic order, but sort to be safe
    results.sort()
    return results

if __name__ == '__main__':
    import sys
    template = sys.stdin.read().strip()
    for pin in solve(template):
        print(pin)
```


```

```


















































