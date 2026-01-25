# DSA: Trie (Prefix Tree)

A Trie is a tree-like data structure for storing strings where each node represents a character.

## 1. Why Trie?
- **Prefix Search**: Find all words starting with "pre" in $O(m)$ where m is prefix length.
- **Autocomplete**: Powering search suggestions.
- **Spell Checking**: Finding similar words.

## 2. Operations
| Operation | Complexity |
|:---|:---:|
| Insert | $O(m)$ |
| Search | $O(m)$ |
| Prefix Search | $O(m)$ |

where $m$ = length of the word/prefix.

## 3. Structure

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_word = False
```
