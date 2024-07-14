# Chemical Formula Parser

This project provides a solution to parse a chemical formula string and count the occurrences of each atom, returning the counts in a specified format.

## Intuition

When parsing a chemical formula, parentheses indicate nested groups of elements which may be multiplied. Using a stack can help manage these nested groups, allowing easy combination and multiplication of element counts as groups close. This stack-based approach helps to maintain scope and manage element counts accurately.

## Approach

1. **Initialize Stack**: Use a stack to handle nested groups of elements. Each dictionary on the stack represents a scope where element counts are stored.
2. **Parse Formula**:
   - Traverse each character in the formula.
   - When encountering '(', push a new dictionary onto the stack.
   - When encountering ')', pop the dictionary from the stack, process the multiplicity, and merge the counts back into the previous scope.
   - When encountering an element, read its full name and count, and update the count in the current scope.
3. **Handle Multiplicities**: After closing parentheses, multiply the counts of elements in that scope by the multiplicity specified.
4. **Merge Counts**: After processing the entire formula, merge all counts and prepare the final output string.
5. **Output Result**: Sort the elements alphabetically and format the result string, appending counts only if they are greater than 1.

## Complexity

- **Time complexity**: The time complexity is \(O(n)\), where \(n\) is the length of the formula. This is because each character in the formula is processed once, and operations such as parsing the element names and counts, as well as stack operations, are efficient.

- **Space complexity**: The space complexity is \(O(n)\), primarily due to the stack used to handle nested groups and the dictionaries storing element counts. In the worst case, the stack size is proportional to the depth of nested parentheses, and the dictionaries hold counts for all elements.

## Code

Here's the C# code to achieve this:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;

public class Solution
{
    public string CountOfAtoms(string formula)
    {
        var stack = new Stack<Dictionary<string, int>>();
        stack.Push(new Dictionary<string, int>());

        int i = 0;
        while (i < formula.Length)
        {
            if (formula[i] == '(')
            {
                // Start a new scope
                stack.Push(new Dictionary<string, int>());
                i++;
            }
            else if (formula[i] == ')')
            {
                // End the current scope
                var top = stack.Pop();
                i++;
                int start = i;
                // Parse the multiplicity after ')'
                while (i < formula.Length && char.IsDigit(formula[i])) i++;
                int multiplicity = i > start ? int.Parse(formula.Substring(start, i - start)) : 1;

                // Merge counts into the previous dictionary on the stack
                foreach (var kv in top)
                {
                    if (!stack.Peek().ContainsKey(kv.Key))
                        stack.Peek()[kv.Key] = 0;
                    stack.Peek()[kv.Key] += kv.Value * multiplicity;
                }
            }
            else
            {
                // Parse element name
                int start = i;
                i++;
                while (i < formula.Length && char.IsLower(formula[i])) i++;
                string name = formula.Substring(start, i - start);

                // Parse multiplicity (count of the atom)
                start = i;
                while (i < formula.Length && char.IsDigit(formula[i])) i++;
                int multiplicity = i > start ? int.Parse(formula.Substring(start, i - start)) : 1;

                // Add or update the count in the current scope
                if (!stack.Peek().ContainsKey(name))
                    stack.Peek()[name] = 0;
                stack.Peek()[name] += multiplicity;
            }
        }

        // Prepare the final result string
        var result = new StringBuilder();
        var finalCount = stack.Pop();
        foreach (var atom in finalCount.OrderBy(kv => kv.Key))
        {
            result.Append(atom.Key);
            if (atom.Value > 1)
                result.Append(atom.Value);
        }

        return result.ToString();
    }
}
