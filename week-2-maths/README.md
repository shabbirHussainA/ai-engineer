# Week 2: Data & Mathematical Foundations for ML

This directory contains the hands-on learning exercises and practice notebooks from Week 2 of my transition from Production Full Stack Engineer to AI Engineer.

## Objective
The primary goal for this week was to build a strong foundation in data manipulation and the mathematical representations underlying Machine Learning, transitioning from raw Python syntax (Week 1) to actual data workflows.

## What I Practiced
During this week, I focused on building intuition and practical skills with standard data science libraries and concepts:

**NumPy & Mathematical Foundations:**
- Arrays, shapes, dimensions, and broadcasting.
- Indexing, slicing, and vectorization techniques.
- Mathematical operations: dot products and matrix multiplication.
- Math/ML intuition: vectors, matrices, mean/median/variance/std, probability intuition, derivatives, gradients, gradient descent, embeddings, and cosine similarity.

**Pandas & Data Manipulation:**
- Working with DataFrames and Series.
- Filtering, sorting, and grouping data (`groupby`).
- Merging and joining datasets.
- Handling missing values and data cleaning.
- Aggregations and CSV manipulation.

## Hands-On Exercises & Strongest Evidence
The strongest piece of technical evidence in this milestone is the missing-data workflow and aggregation exercise in the Pandas notebook, which includes:
- Detecting missingness in a dataset and inspecting affected rows.
- Filling missing `age` values using the mean and `income` using the median.
- Calculating metrics via `groupby`: customer count, average age, average spend, and churn rate.

> **Note:** These are learning exercises designed to build fundamental skills. They are not intended to represent production ML pipelines, novel architectures, or sophisticated data science projects.

## Repository Structure
- `pandas.ipynb` - Data manipulation exercises (filtering, groupby, missing data, etc.).
- `Numpy/` - Notebooks focused on NumPy arrays, matrix math, and vectorized operations.

## Limitations
- These notebooks use synthetic or basic learning datasets, not large-scale production data.
- The focus is entirely on data preparation and mathematical fundamentals, without building predictive models yet.
- Metrics are calculated for learning purposes rather than business reporting.

## Next Steps
These fundamentals directly connect to later ML work. Understanding how data is represented mathematically (tensors/matrices) and how to manipulate it (Pandas/NumPy) is a prerequisite for feeding data into machine learning models, calculating loss, and optimizing weights via gradient descent in the upcoming weeks.
