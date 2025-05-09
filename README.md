# Custom Plugin: Ollama API Integration in Dataiku
![Logo Ollama](https://avatars.githubusercontent.com/u/151674099?s=48&v=4) + ![Logo Dataiku](https://avatars.githubusercontent.com/u/2335170?s=48&v=4)

This Dataiku custom plugin allows you to leverage the power of the ) [Ollama](https://ollama.ai/) API directly within your  [Dataiku](https://www.dataiku.com/) workflows. You can process a column of text data using a locally running Ollama model to generate content based on a free-form prompt and automatically extract structured information from the model's JSON output.

## Overview

The plugin provides a Python recipe that takes an input dataset, a text column to process, and instructions for the Ollama model. It sends each row's text to the specified Ollama model, which generates a response according to your prompt. The plugin is specifically designed to work with Ollama's JSON output format, allowing you to define the expected structure of the generated content using a Pydantic-style definition. The recipe then parses this JSON output and adds the extracted fields as new columns to your output dataset.

**Key Features:**

* **Seamless Ollama Integration:** Connects directly to your locally running Ollama instance.
* **Free-Form Prompting:** Utilize the full flexibility of language models with custom system roles and instructions.
* **Structured Data Extraction:** Define the expected JSON output format using a user-friendly Pydantic-like syntax.
* **Automatic Schema Inference:** The plugin uses your Pydantic definition to understand and extract fields from the Ollama response.
* **Concurrency Control:** Manage the number of simultaneous calls to your Ollama API to optimize performance and resource usage.
* **Robust Error Handling:** Provides informative error messages and includes metadata about the processing.
* **Flexible Output:** Choose to keep only the original columns and the extracted JSON fields or include additional metadata like model ID, processing date, and performance metrics.

## Important Considerations

* **Ollama Setup:** Ensure that Ollama is running correctly on your local machine and is accessible at `http://localhost:11434`.
* **Model Availability:** The Ollama model ID you specify must be a model that you have already pulled using `ollama pull`.
* **JSON Output:** The effectiveness of this plugin relies on the Ollama model consistently producing output that can be parsed as JSON according to your Pydantic definition. Fine-tuning your prompts can be crucial for achieving the desired structured output.
* **Performance:** The processing time will depend on the size of your dataset, the complexity of your prompts, and the performance of your local machine running Ollama. The concurrency control parameter can help manage resource usage.
* **Error Handling:** While the plugin includes error handling, it's important to monitor the output dataset and the Dataiku recipe logs for any processing errors.
* **OS Requirement:** This plugin is specifically designed and tested for use with Dataiku running on a [Linux](https://www.dataiku.com/product/get-started/linux/) or [Mac](https://www.dataiku.com/product/get-started/mac/) where Ollama is installed locally.

## Installation

This plugin is designed to be installed within your Dataiku instance. Follow these steps:

1.  **Prerequisites:**
    * **Dataiku DSS:** Ensure you have a working Dataiku Data Science Studio instance (version 12.0 or later recommended).
    * **Ollama:** You need to have [Ollama](https://ollama.ai/) installed and running on your **Linux local machine** or **Mac** where the Dataiku agent executes Python code. The plugin communicates with Ollama via its default API endpoint (`http://localhost:11434`).
    * **Ollama Models:** Make sure you have downloaded the Ollama model(s) you intend to use with this plugin (e.g., `llama3:latest`, `gemma2:2b`). You can do this using the `ollama pull <model_id>` command in your terminal.

2.  **Plugin Installation:**
    * In your Dataiku DSS instance, navigate to **Plugins** in the top navigation bar.
    * Click on **+ Add a plugin** in the top right corner.
    * Select **Upload a plugin**.
    * Browse to the plugin's ZIP file (once you have it) and upload it.
    * Follow the on-screen instructions to install the plugin.

## How to Use

Once the plugin is installed, you can use the "OLLAMA API" recipe in your Dataiku flows:

1.  **Create a New Recipe:** In your Dataiku flow, click on the **+** icon to add a new step. Navigate to **Plugins** and select **OLLAMA API**.

2.  **Input Dataset:** Select the input dataset that contains the text column you want to process.

3.  **Output Dataset:** Choose or create the output dataset where the processed data and extracted fields will be written.

4.  **Configure the Recipe:** In the recipe's settings, you will find the following parameters:

    * **Input Column Name:** Specify the name of the column in your input dataset that contains the text you want to send to the Ollama model.

    * **System Prompt - Role:** (Optional) Define a role or persona for the Ollama model (e.g., "You are a helpful assistant specializing in concise summaries."). The default is "You are a helpful assistant."

    * **System Prompt - Main Instruction:** Define the primary task you want the Ollama model to perform based on the input column content (e.g., "Extract key entities from the following text."). The default is "Explain in one sentence."

    * **Expected JSON Output Fields (Pydantic-style):** This is a crucial parameter. Here, you define the structure of the JSON output you expect from the Ollama model. You list the fields line by line using a syntax similar to Pydantic model field definitions.

        * Lines starting with `#` are treated as comments.
        * For each field, specify the name, the Python type, and optionally a `Field()` with a description and other Pydantic constraints.
        * **Examples:**
            ```
            objective: str = Field(description='The main goal of the sport.')
            key_actions: List[str] = Field(description='A list of main actions.')
            number_of_players: Optional[int] = Field(default=None, description='Number of players.')
            is_outdoor: Optional[bool] = Field(default=None, description='True if played outdoors.')
            main_equipment: Optional[Dict[str, str]] = Field(default=None, description='A dictionary of the main equipment used, with equipment name and a brief description.')
            ```
        * The plugin uses this definition to generate a JSON schema for the prompt and to validate and extract data from the Ollama response.

    * **Ollama Model ID:** Enter the ID of the Ollama model you want to use (e.g., `llama3:latest`, `gemma2:2b`). Use the `ollama list` command in your terminal to see the available models. The default is `gemma2:2b`.

    * **Temperature:** (Optional) Control the randomness of the model's output. Lower values (e.g., 0.0 to 0.3) are less random and more deterministic, often preferred for structured data extraction. Higher values (e.g., 0.7 to 1.0) lead to more creative or varied responses. The default is `0.0`.

    * **Model Keep Alive Duration:** (Optional) Controls how long the model stays loaded in memory on the Ollama server after a request. Examples: `5m`, `30s`, `0` (unload immediately), `-1` (keep loaded indefinitely). Leaving it empty uses the Ollama default (usually 5 minutes).

    * **Advanced Ollama API Options (JSON):** (Optional) Provide advanced options for the Ollama API call as a JSON object string (e.g., `{"seed": 123, "num_predict": -1}`). Note that the `Temperature` and `Keep Alive Duration` parameters above will override corresponding keys if also present in this JSON. The default is `{}`.

    * **Refine Output Columns:** (Optional) If selected (True), the output dataset will only contain the original input columns plus the columns derived from the "Expected JSON Output Fields." If unselected (False), additional metadata columns (model ID, system prompt, processing date, error messages, and performance metrics) will also be included. The default is `True`.

5.  **Run the Recipe:** Once you have configured the recipe, click on the **Run** button to process your data.

## Example Usage

Let's say you have a dataset with a column named `Description` containing text descriptions of various sports. You want to use an Ollama model to identify the sport being described and output the result in a structured JSON format.

**Input (sports.csv):**

| Description | Sport_truth |
|-------------|-------------|
| The ball is round and often black and white. Players use their feet and heads, but rarely their hands. Two teams compete to score by getting the ball into a net. | Football (Soccer) |
| The court is divided by a net. Players use rackets to hit a felt-covered ball. Points are scored when the opponent fails to return the ball legally. | Tennis |
| Participants glide across a frozen surface. Blades attached to boots allow for movement. Speed and precision are highly valued. | Figure Skating |
| Two teams of players face off on a rink. They use curved sticks to maneuver a hard rubber disc. The objective is to shoot the disc into the opponent's net. | Ice Hockey |
| Athletes propel themselves through water. Different strokes are used for various events. Races are held in lanes within a pool. | Swimming |
| Players hit a small white ball with clubs. The aim is to get the ball into a series of holes. The course consists of fairways, greens, and hazards. | Golf |
| Two opponents try to pin each other to the mat. Grappling and submission holds are key techniques. Matches take place in a circular area. | Wrestling |
| Athletes run on a track or cross-country terrain. Distances vary from sprints to marathons. Timing is crucial for measuring performance. | Track and Field (Running) |
| Participants ride on horseback. Events include dressage, show jumping, and eventing. Communication and partnership with the animal are crucial. | Equestrianism |
| Two teams throw a ball through a hoop. Players advance the ball by dribbling and passing. The court is rectangular with a hoop at each end. | Basketball |

**Recipe Configuration:**

* **Input Column Name:** `Description`
* **System Prompt - Role:** `You are an expert in sport.`
* **System Prompt - Main Instruction:** `Analyze the following description of a sport to determine it and identify key features. Extract information about the sport's objective, key actions involved, typical number of players, average duration, playing environment (indoor/outdoor), main equipment used, and common penalties.`
* **Expected JSON Output Fields (Pydantic-style):**
    ```
    identified_sport: str = Field(description='The name of the identified sport.')
    objective: str = Field(description='The main goal or objective of the sport.')
    key_actions: List[str] = Field(description='A list of the main actions or movements performed by participants.')
    number_of_players: Optional[int] = Field(default=None, description='The typical number of players involved.')
    average_duration_minutes: Optional[float] = Field(default=None, description='The average duration of a game in minutes.')
    is_outdoor: Optional[bool] = Field(default=None, description='True if the sport is usually played outdoors, False otherwise.')
    main_equipment: Optional[Dict[str, str]] = Field(default=None, description='A dictionary of the main equipment used, with equipment name and a brief description.')
    common_penalties: Optional[List[str]] = Field(default=None, description='A list of common penalties or rule violations.')
    ```
* **Ollama Model ID:** `qwen3:4b` (assuming you have this model pulled)
* **Temperature:** `0.1` (for more deterministic results)
* **Refine Output Columns:** `True`

**Output Dataset (Example):**

The output dataset will contain the original `Description` column and the newly extracted `identified_sport` and `confidence_level` columns.

| Description | Sport_truth | Description_identified_sport | Description_objective | Description_key_actions | Description_number_of_players | Description_average_duration_minutes | Description_is_outdoor | Description_main_equipment | Description_common_penalties | Description_processing_error | Description_ollama_total_duration_ns | Description_ollama_load_duration_ns | Description_ollama_prompt_eval_duration_ns | Description_ollama_eval_duration_ns | Description_ollama_prompt_eval_count | Description_ollama_eval_count | Description_ollama_done_reason |
|-------------|-------------|-------------------------------|-----------------------|-------------------------|--------------------------------|---------------------------------------|------------------------|-------------------------------|--------------------------------|-------------------------------|----------------------------------------|---------------------------------------|-----------------------------------------------|---------------------------------------|----------------------------------------|----------------------------------------|-------------------------------|
| The ball is round and often black and white.Players use their feet and heads, but rarely their hands.Two teams compete to score by getting the ball into a net.Matches are played on a large rectangular field of grass.Referees enforce the rules of the game.Injuries like sprains and tackles are not uncommon.Fans cheer loudly from the stands.World championships are held every four years.Famous athletes become global icons.The offside rule can be quite controversial. | Football (Soccer) | Football | To score more goals than the opposing team by getting the ball into their net | ['Kicking the ball', 'Heading the ball', 'Passing the ball', 'Tackling opponents', 'Running with the ball'] | 11 | 90.0 | True | {'key_main_equipment': 'Football'} | ['Offside', 'Fouls', 'Yellow card', 'Red card'] |  | 10123659625 | 33118292 | 1032338000 | 9055890792 | 359 | 152 | stop |
| The court is divided by a net.Players use rackets to hit a felt-covered ball.Points are scored when the opponent fails to return the ball legally.Matches can be singles or doubles.Different surfaces like clay, grass, and hard court affect the game.Tiebreakers can lead to tense moments.Grand Slam tournaments are highly prestigious.Serving involves tossing the ball and hitting it over the net.Foot faults can result in losing a serve.The scoring system uses terms like 'love', 'deuce', and 'advantage'. | Tennis | Tennis | To win a match by scoring points, which are determined by the ability to return the ball legally and force the opponent into a fault. | ['Hitting the ball with a racket over the net', 'Serving the ball from the baseline', 'Returning the ball after it crosses the net', 'Avoiding foot faults during serves', 'Executing tiebreakers during matches', 'Scoring points through valid returns and faults by the opponent'] | 2 | 2.5 | True | {'key_main_equipment': 'Tennis racket'} | ['Foot fault during serve', 'Faulty return of the ball', 'Loss of serve due to a fault', 'Tiebreaker rules enforcement'] |  | 13563005292 | 34865084 | 2051624792 | 11473023292 | 372 | 211 | stop |
| Participants glide across a frozen surface.Blades attached to boots allow for movement.Speed and precision are highly valued.Competitions can involve jumps, spins, and intricate footwork.Pairs and synchronized events are common.Music often accompanies performances.Falls can occur, requiring grace and recovery.Olympic medals are a major achievement.Rinks provide the playing area.Costumes are often elaborate and thematic. | Figure Skating | Figure Skating | To perform the most aesthetically pleasing and technically proficient routine to music, with the highest level of skill and artistry. | ['Gliding across ice', 'Executing jumps and spins', 'Performing intricate footwork', 'Synchronized movements', 'Interpreting music'] | 2 | 2.5 | False | {'key_main_equipment': 'Ice skates with blades'} | ['Falls during performance', 'Technical errors in routines'] |  | 9032802958 | 11936375 | 307193833 | 8711499875 | 344 | 161 | stop |
| Two teams of players face off on a rink.They use curved sticks to maneuver a hard rubber disc.The objective is to shoot the disc into the opponent's net.Goalies wear specialized protective equipment.Physical contact and checking are part of the game.Periods divide the match into segments.Overtime can occur if the score is tied.Fans often chant and cheer loudly.The Stanley Cup is the ultimate prize.Power plays and penalty kills are strategic elements. | Ice Hockey | Hockey | To shoot the disc into the opponent's net | ['Using curved sticks to maneuver a hard rubber disc', Shooting the disc into the opponent's net, 'Physical contact and checking', 'Periods dividing the match into segments', 'Overtime if the score is tied', 'Power plays and penalty kills'] | 18 | 60.0 | False | {'key_main_equipment': 'Curved sticks and a hard rubber disc'} | ['Physical contact', 'Checking', 'Tied score leading to overtime'] |  | 9817760291 | 12480750 | 309640708 | 9493579500 | 356 | 176 | stop |
| Athletes propel themselves through water.Different strokes are used for various events.Races are held in lanes within a pool.Timing is crucial for recording personal bests.World records are constantly being challenged.Goggles and caps are common pieces of equipment.Training involves intense physical conditioning.Relay events involve teams of four.Starts from blocks require explosive power.Breathing techniques are essential for endurance. | Swimming | Swimming | To swim a certain distance or time faster than other competitors | ['Propelling through water using arms and legs', 'Using different strokes such as freestyle, butterfly, and breaststroke', 'Maintaining a steady rhythm and breathing pattern', 'Competing in lanes with specific race times', 'Participating in relay races with baton exchanges'] | 4 | 45.0 | False | {'key_main_equipment': 'Swim gear'} | ['Touching the wall before finishing the race', 'Entering the water before the start', 'Violating the starting rules'] |  | 10302410875 | 12156708 | 305466458 | 9982610042 | 343 | 184 | stop |
| Players hit a small white ball with clubs.The aim is to get the ball into a series of holes.The course consists of fairways, greens, and hazards.Scoring is based on the number of strokes taken.Etiquette and rules are strictly observed.Caddies often assist players with strategy and equipment.Tournaments are held over multiple days.Weather conditions can significantly impact the game.Different types of clubs are used for varying distances.A good swing requires precision and timing. | Golf | Golf | To get the ball into the hole in the fewest number of strokes possible | ['Swing the club to hit the ball', 'Navigate the course through fairways and hazards', 'Use different clubs for varying distances', 'Approach the green with precision shots', 'Maintain proper etiquette and follow rules'] | 4 | 4.5 | True | {'key_main_equipment': 'Golf club'} | ['Wrong ball used', 'Incorrect stroke', 'Unlawful delay', 'Bunkers or water hazards'] |  | 10005097750 | 13197834 | 373960834 | 9615732625 | 360 | 178 | stop |
| Two opponents try to pin each other to the mat.Grappling and submission holds are key techniques.Matches take place in a circular area.Weight classes ensure fair competition.Throws and takedowns can lead to victory.Referees oversee the match and award points.Training emphasizes strength, agility, and technique.Olympic and world championships are major events.Different styles exist, each with its own rules.Respect for the opponent is a core principle. | Wrestling | Judo | To pin the opponent to the mat or score enough points through throws and takedowns to win the match. | ['Grappling', 'Submission holds', 'Throws', 'Takedowns', 'Pinning the opponent'] | 2 | 5.0 | False | {'key_main_equipment': 'Gi (a uniform consisting of a jacket and pants)'} | ['Using illegal holds', 'Striking an opponent', 'Touching the head or neck', 'Unsportsmanlike behavior'] |  | 9628890000 | 12861250 | 312146250 | 9301512875 | 355 | 172 | stop |
| Athletes run on a track or cross-country terrain.Distances vary from sprints to marathons.Timing is crucial for measuring performance.Spiked shoes provide traction.Endurance and speed are essential qualities.Relay races involve teams passing a baton.Olympic and world championships showcase top talent.Training regimens are often rigorous.Nutrition and recovery play vital roles.Finishing strong is the ultimate goal. | Track and Field (Running) | Track and Field | To achieve the best performance in running events by maximizing speed, endurance, and precision in timing. | ['Running on a track or cross-country course', 'Competing in sprints, marathons, and relay races', 'Using spiked shoes for traction and performance', 'Maintaining precise timing during competitions', 'Participating in relay races with baton exchanges'] | 2 | 4.5 | True | {'key_main_equipment': 'Spiked running shoes'} | ['Battling with another runner', 'Failing to maintain proper form', 'Breaking the rules during a race'] |  | 10631634875 | 12558292 | 309463375 | 10307524875 | 349 | 191 | stop |
| Participants ride on horseback.Events include dressage, show jumping, and eventing.Communication and partnership with the animal are crucial.Skill and elegance are often displayed.Competitions take place in arenas or open fields.Different breeds of horses are suited for various disciplines.Training requires patience and understanding.Safety for both rider and horse is paramount.Equipment includes saddles, bridles, and helmets.The bond between human and animal is evident. | Equestrianism | Equestrian sports | To demonstrate skill, control, and partnership with a horse through various disciplines such as dressage, show jumping, and eventing. | ['Riding on horseback', 'Performing dressage routines', 'Participating in show jumping events', 'Engaging in eventing competitions', 'Communicating with and controlling the horse'] | 2 | 60.0 | True | {'key_main_equipment': 'Saddles, bridles, and helmets'} | ['Falling off the horse', 'Incorrect use of equipment', 'Violation of competition rules'] |  | 10253142166 | 12072416 | 314827875 | 9924186958 | 354 | 184 | stop |
| Two teams throw a ball through a hoop.Players advance the ball by dribbling and passing.The court is rectangular with a hoop at each end.Points are scored based on the shot's distance.Fouls can result in free throws.Games are divided into quarters.Fast breaks and set plays are key strategies.Rebounding is crucial for gaining possession.Famous players become cultural icons.The sound of the swishing net is satisfying. | Basketball | Basketball | To score more points than the opposing team by shooting a ball through a hoop. | ['Dribbling', 'Passing', 'Shooting', 'Rebounding', 'Fast breaks', 'Set plays'] | 5 | 48.0 | False | {'key_main_equipment': 'Basketball'} | ['Fouls', 'Free throws', 'Disqualification'] |  | 6959494416 | 12924125 | 309363125 | 6635114167 | 349 | 145 | stop |

## Contributing

Contributions to this plugin are welcome! Please feel free to submit pull requests or open issues on the associated GitHub repository.

