---
title: "The Addictive Power of C# - is { } Pattern"
date: 2025-08-11
tags:
- "CSharp"
- "Patterns"
comments: true
---

The `is { }` pattern in C# is incredibly addictive once you discover it. It's such a clean way to check for non-null and simultaneously assign to a non-nullable variable.

The beauty of the `is { }` pattern is that it does two things at once:

1. **Null check**: Verifies the variable isn't null
2. **Smart cast**: Creates a new non-nullable variable you can use immediately

What makes it so addictive is how it eliminates the cognitive overhead of traditional null checking. Instead of:
```csharp
if (nullableVariable != null)
{
    // Still need to use nullableVariable with ! or pragma warnings
    var result = nullableVariable.SomeMethod();
}
```

You get:

```csharp
if (nullableVariable is { } notNullable)
{
    // Compiler knows notNullable is definitely not null
    var result = notNullable.SomeMethod();
}
```

The pattern becomes even more powerful when combined with property patterns, destructuring, and guard clauses. It's one of those features that once you start using, you find yourself reaching for it everywhere because it makes the code both safer and more readable!

Here are some cool examples focusing specifically on this pattern:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// Example 1: Basic null check with immediate use
string? nullableText = GetUserInput();
if (nullableText is { } text && text.Length > 5)
{
    Console.WriteLine($"Valid input: {text.ToUpper()}");
}

// Example 2: Chaining with property access
Person? person = FindPerson(id);
if (person is { } p && p.Address is { } addr && addr.City.Length > 0)
{
    Console.WriteLine($"Person lives in {addr.City}");
}

// Example 3: Collection handling
List<string>? items = GetItems();
if (items is { Count: > 0 } list)
{
    var first = list.First(); // No null warnings!
    Console.WriteLine($"First item: {first}");
}

// Example 4: Multiple nullable checks in one condition
string? name = GetName();
int? age = GetAge();
if (name is { } n && age is { } a && a >= 18)
{
    Console.WriteLine($"{n} is an adult ({a} years old)");
}

// Example 5: Dictionary operations
Dictionary<string, object>? config = LoadConfig();
if (config is { } cfg && cfg.TryGetValue("timeout", out var timeoutObj) 
    && timeoutObj is int timeout)
{
    SetTimeout(timeout);
}

// Example 6: Nested object access
Response? apiResponse = await CallApi();
if (apiResponse is { Data: { } data } response && 
    data.Results is { Count: > 0 } results)
{
    ProcessResults(results);
}

// Example 7: Method chaining with null checks
string? input = Console.ReadLine();
if (input is { } i && i.Trim() is { Length: > 0 } trimmed)
{
    var processed = trimmed.ToLowerInvariant();
    // No more null reference warnings!
}

// Example 8: Event handling
EventHandler<DataEventArgs>? handler = GetEventHandler();
DataEventArgs? eventArgs = CreateEventArgs();
if (handler is { } h && eventArgs is { } args)
{
    h.Invoke(this, args); // Both guaranteed non-null
}

// Example 9: Complex pattern matching with properties
User? user = GetCurrentUser();
if (user is { IsActive: true, Permissions: { } perms } u && 
    perms.Contains("admin"))
{
    ShowAdminPanel(u);
}

// Example 10: Array/span operations
int[]? numbers = GetNumbers();
if (numbers is { Length: > 5 } nums)
{
    var average = nums.Average(); // No null check needed
    Console.WriteLine($"Average of {nums.Length} numbers: {average}");
}

// Example 11: String operations with immediate pattern matching
string? jsonString = GetJsonData();
if (jsonString is { } json && 
    System.Text.Json.JsonDocument.Parse(json) is { } doc)
{
    // Both json and doc are guaranteed non-null
    ProcessJsonDocument(doc);
}

// Example 12: Combining with switch expressions
string? ProcessData(object? data) => data switch
{
    string s when s is { Length: > 0 } => $"String: {s}",
    int[] arr when arr is { Length: > 0 } nums => $"Array with {nums.Length} items",
    Person p when p is { Name: { } name } => $"Person: {name}",
    _ => "Unknown or null data"
};

// Example 13: File operations
FileInfo? file = GetFileInfo(path);
if (file is { Exists: true } f && f.Length > 0)
{
    var content = File.ReadAllText(f.FullName);
    ProcessContent(content);
}

// Example 14: Database-style operations
var query = from item in GetItems()
            where item is { IsValid: true, Value: { } val } 
                  && val > threshold
            select new { item.Id, Value = val };

// Example 15: Exception handling with patterns
Exception? lastException = GetLastException();
if (lastException is { InnerException: { } inner } ex)
{
    LogError($"Outer: {ex.Message}, Inner: {inner.Message}");
}

// Bonus: The pattern works great with readonly fields too!
private readonly string? _cachedValue;

public void UseCache()
{
    if (_cachedValue is { } cached)
    {
        // cached is definitely not null here
        ProcessValue(cached.ToUpper());
    }
}

// Helper classes for examples
public class Person
{
    public string? Name { get; set; }
    public Address? Address { get; set; }
    public bool IsActive { get; set; }
    public List<string>? Permissions { get; set; }
}

public class Address
{
    public string City { get; set; } = "";
}

public class Response
{
    public ResponseData? Data { get; set; }
}

public class ResponseData
{
    public List<object>? Results { get; set; }
}

public class User : Person { }

public class DataEventArgs : EventArgs
{
    public object? Data { get; set; }
}
```
