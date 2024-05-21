# Movie Review Summarizer

This project aims to summarize movie reviews using AI and display the summaries via a Gradio interface. Users can select a movie from a dropdown menu and view the summarized reviews, including the overall sentiment and notable opinions.

## Features
- **Data Processing**: Preprocesses and summarizes movie reviews.
- **Summarization**: Uses OpenAI's GPT-4 model to generate comprehensive summaries.
- **User Interface**: Provides an easy-to-use Gradio interface to access the summaries.

## Requirements
- Python 3.6 or later
- Pandas
- OpenAI
- Gradio

## Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/sir-hobbyjog/Review_analysis.git
   cd moviereview-summarizer
   ```

2. Install the required packages:
   ```bash
   pip install pandas openai gradio
   ```

3. Place your movie reviews dataset in the project directory. The dataset should be a CSV file with columns `movie_title` and `review`.

## Setup
1. **Summarize Reviews**:
   Run the summarization script to generate summaries for each movie and save them in a JSON file.
   ```python
   import openai
   import pandas as pd
   import json

   # Initialize OpenAI API
   openai.api_key = 'your_openai_api_key'

   # Load your dataset
   df = pd.read_csv('movie_reviews.csv')

   # Group reviews by movie
   grouped_reviews = df.groupby('movie_title')['review'].apply(list).reset_index()

   def summarize_reviews(reviews):
       reviews_text = ' '.join(reviews)
       messages = [
           {"role": "system", "content": "You are a helpful assistant."},
           {"role": "user", "content": (
               "Summarize the following movie reviews. "
               "First, state what the majority of viewers are saying. "
               "Second, mention what the minority of viewers are saying. "
               "Finally, provide the overall sentiment of the reviews:\n\n"
               f"{reviews_text}"
           )}
       ]
       response = openai.ChatCompletion.create(
           model="gpt-4",
           messages=messages,
           max_tokens=150
       )
       summary = response.choices[0].message['content'].strip()
       return summary

   # Create a dictionary to store summaries
   movie_summaries = {}

   # Iterate over each movie and summarize the reviews
   for index, row in grouped_reviews.iterrows():
       movie_title = row['movie_title']
       reviews = row['review']
       summary = summarize_reviews(reviews)
       movie_summaries[movie_title] = summary
       print(f"Summarized reviews for {movie_title}")

   # Save the summaries to a JSON file
   with open('movie_summaries.json', 'w') as f:
       json.dump(movie_summaries, f)

   print("Summarization complete. Summaries saved to movie_summaries.json")
   ```

2. **Run the Gradio Interface**:
   Use the following script to create and launch the Gradio interface:
   ```python
   import gradio as gr
   import json

   # Load the summaries from the JSON file
   with open('movie_summaries.json', 'r') as f:
       movie_summaries = json.load(f)

   def get_movie_summary(movie_title):
       summary = movie_summaries.get(movie_title, "Sorry, no summary available for this movie.")
       return summary

   movie_titles = list(movie_summaries.keys())

   title_dropdown = gr.inputs.Dropdown(choices=movie_titles, label="Select a movie title")
   summary_output = gr.outputs.Textbox(label="Movie Summary")

   interface = gr.Interface(fn=get_movie_summary,
                            inputs=title_dropdown,
                            outputs=summary_output,
                            title="Movie Review Summarizer",
                            description="Select a movie title to get a summary of reviews.")

   interface.launch()
   ```

## Usage
1. Summarize the reviews by running the summarization script.
2. Launch the Gradio interface to interact with the summaries.

## Acknowledgements
- OpenAI for providing the GPT-4 model.
- Gradio for the easy-to-use interface library.

## License
We don't need no stinking badges
