# Movie Score Prediction and Generative AI Insights

## Introduction
This project aims to predict movie scores/user ratings based on genre and runtime using BigQuery ML and then generate insights into other factors influencing movie scores using Generative AI with the Vertex AI PaLM API. We utilize the MovieLens dataset and Google Cloud services for this purpose.

### Dataset
The dataset used for this project is derived from the MovieLens dataset, specifically the [movies_data.csv](https://github.com/AbiramiSukumaran/movie_score_genai_insights/blob/main/movies_data.csv) file obtained from [movielens](https://grouplens.org/datasets/movielens/1m/) source.

## Data to ML
1. **Movies Dataset**: Stored in BigQuery.
2. **Movie Score Prediction Model**: Trained with BigQuery ML using logistic regression, considering genre and runtime.
3. **SQL-only Prediction**: User rating prediction and result analysis conducted using SQL queries.
4. **Model Deployment**: Deployed the model in Vertex AI for a REST endpoint.
5. **Storing Prediction Results**: Stored prediction results in BigQuery for further analysis.

### Steps
Follow the [codelab](https://codelabs.developers.google.com/moviescore-prediction-bqmlsql), with modifications in Step 6 and prediction query.

#### Model Creation Query
```sql
CREATE OR REPLACE MODEL
  `movies.movies_score_model`
OPTIONS
  ( model_type='LOGISTIC_REG',
    auto_class_weights=TRUE,
    data_split_method='NO_SPLIT',
    input_label_cols=['score']
  ) AS
SELECT name, genre,runtime, score
FROM
  movies.movies_score
WHERE
  data_cat = 'TRAIN';



  # Prediction Query

CREATE TABLE movies.predicted_movies_score as (
SELECT
  *
FROM
  ML.PREDICT (MODEL movies.movies_score_model,
    (
    SELECT
      *
    FROM
      movies.movies_rating
    WHERE
      data_cat= 'TEST'
     )
  )
);


Data to Generative AI
Analysis of Factors: Besides genre and runtime, other factors influencing movie ratings are analyzed using Generative AI with text-bison model via SQL queries.
BigQuery PaLM API Integration: Utilize BigQuery's GENERATE_TEXT construct to invoke the PaLM API from Vertex AI.
External Connection Setup: Establish the access between BigQuery ML and Vertex services.

Steps
Follow the codelab for Data to Generative AI, starting from Step 6, with modifications in steps 8 and 10.

Model Creation Query
CREATE OR REPLACE MODEL
  movies.llm_model REMOTE
WITH CONNECTION `us-central1.bq_llm_connection` OPTIONS (remote_service_type = 'CLOUD_AI_LARGE_LANGUAGE_MODEL_V1');


PaLM API Query
SELECT
  *
FROM
  ML.GENERATE_TEXT( MODEL movies.llm_model,
    (
    SELECT
      CONCAT('FROM THE FOLLOWING TEXT ABOUT MOVIES, WHAT DO YOU THINK ARE THE FACTORS INFLUENCING A MOVIE SCORE TO BE GREATER THAN 5?: ', movie_data) AS prompt
    FROM (
      SELECT
        REPLACE(STRING_AGG( CONCAT('A movie named ',name, ' from the country ', country, ' with a censor rating of ',rating, ' and a budget of ', budget, ' produced by ', company, ' with a runtime of about ', runtime, ' and in the genre ', genre, ' starring ', star, ' has had a success score of ', score, '') ), ',','. ') AS movie_data
      FROM (
        SELECT
          *
        FROM
          `abis-345004.movies.movies_rating`
        WHERE
          CAST(SCORE AS INT64) > 5
        LIMIT
          50) ) AS MOVIES),
    STRUCT( 0.2 AS temperature,
      100 AS max_output_tokens,
      TRUE AS flatten_json_output));



Result Summary
Based on the given information, the following factors seem to influence a movie score to be greater than 5:

Genre: All the movies with a success score greater than 5 belong to the crime genre.
Censor Rating: All the movies with a success score greater than 5 have a censor rating of R.
Author: Dhiraj Chaudhari