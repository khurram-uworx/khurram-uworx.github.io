---
title: "Predicting Task Time"
date: 2025-06-07
tags:
- ML
- "ML.NET"
---

This post introduces Linear Regression in a way that's tailored for software engineers—practical, intuitive, and surprisingly useful. Let’s turn your task estimates from guesswork into data-driven decisions.

### Problem Identification

A relatable real-world problem for software engineers that can be solved using regression is **predicting the time required to complete a software development task** (e.g., fixing a bug, implementing a feature, or writing a module). This is often referred to as **effort estimation** or **task duration prediction**. Accurate predictions can help teams plan better, allocate resources efficiently, and meet deadlines.

### Why Regression?
Regression is suitable because:
1. The output (time to complete a task) is a continuous value.
2. Features like task complexity, developer experience, lines of code, and dependencies can influence the prediction.

---

### Solution Implementation Using ML.NET

Let’s implement a simple regression model in ML.NET to predict the time required to complete a software development task.

#### Step 1: Install ML.NET
First, install the ML.NET NuGet package in your .NET project:
```bash
dotnet add package Microsoft.ML
```

#### Step 2: Define the Data Model
Create a class to represent the input data and the prediction.

```csharp
using Microsoft.ML.Data;

public class TaskData
{
    [LoadColumn(0)] public float Complexity; // Task complexity (e.g., 1 to 10)
    [LoadColumn(1)] public float DeveloperExperience; // Developer experience in years
    [LoadColumn(2)] public float LinesOfCode; // Estimated lines of code
    [LoadColumn(3)] public float Dependencies; // Number of dependencies
    [LoadColumn(4)] public float ActualTime; // Actual time taken (in hours)
}

public class TaskTimePrediction
{
    [ColumnName("Score")]
    public float PredictedTime; // Predicted time in hours
}
```

#### Step 3: Load and Prepare Data
Assume you have a CSV file (`tasks.csv`) with historical task data:

```csv
Complexity,DeveloperExperience,LinesOfCode,Dependencies,ActualTime
5,2,100,3,8
7,5,300,5,20
3,1,50,1,4
8,3,400,6,25
...
```

Load and prepare the data in ML.NET:

```csharp
using Microsoft.ML;
using Microsoft.ML.Data;

var mlContext = new MLContext();

// Load data from CSV
IDataView dataView = mlContext.Data.LoadFromTextFile<TaskData>("tasks.csv", hasHeader: true, separatorChar: ',');

// Split data into training and test sets
var splitData = mlContext.Data.TrainTestSplit(dataView, testFraction: 0.2);
```

#### Step 4: Define and Train the Model
Use a regression algorithm (e.g., FastTree) to train the model.

```csharp
// Define data preparation and model pipeline
var pipeline = mlContext.Transforms.Concatenate("Features", 
                nameof(TaskData.Complexity), 
                nameof(TaskData.DeveloperExperience), 
                nameof(TaskData.LinesOfCode), 
                nameof(TaskData.Dependencies))
    .Append(mlContext.Regression.Trainers.FastTree(labelColumnName: nameof(TaskData.ActualTime), featureColumnName: "Features"));

// Train the model
var model = pipeline.Fit(splitData.TrainSet);
```

#### Step 5: Evaluate the Model
Evaluate the model's performance on the test set.

```csharp
var predictions = model.Transform(splitData.TestSet);
var metrics = mlContext.Regression.Evaluate(predictions, labelColumnName: nameof(TaskData.ActualTime));

Console.WriteLine($"R^2: {metrics.RSquared}");
Console.WriteLine($"Mean Absolute Error: {metrics.MeanAbsoluteError}");
Console.WriteLine($"Mean Squared Error: {metrics.MeanSquaredError}");
```

#### Step 6: Make Predictions
Use the trained model to predict the time for a new task.

```csharp
var predictionEngine = mlContext.Model.CreatePredictionEngine<TaskData, TaskTimePrediction>(model);

var newTask = new TaskData
{
    Complexity = 6,
    DeveloperExperience = 4,
    LinesOfCode = 200,
    Dependencies = 4
};

var prediction = predictionEngine.Predict(newTask);
Console.WriteLine($"Predicted Time: {prediction.PredictedTime} hours");
```

---

### Example Output
For a task with:
- Complexity = 6
- Developer Experience = 4 years
- Lines of Code = 200
- Dependencies = 4

The model might predict:
```
Predicted Time: 12.5 hours
```

---

### Improvements
1. **Feature Engineering**: Add more features like task type (bug, feature, refactor), team size, or priority level.
2. **Data Collection**: Use historical data from project management tools (e.g., Jira, Trello).
3. **Advanced Algorithms**: Experiment with other regression algorithms like LightGBM or SGD.

This solution provides a practical way for software teams to estimate task durations using ML.NET!