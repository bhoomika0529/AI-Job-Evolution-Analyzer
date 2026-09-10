## AI Job Evolution Analyzer

AI Job Evolution Analyzer is a project that looks at how AI can affect different parts of a job instead of simply saying that a whole job will be replaced.

The main idea is that a job consists of many different tasks. Some tasks can be automated more easily, some can be supported by AI, and some still depend heavily on human judgement, communication, responsibility or physical work.

This project takes a job role as input and analyses the tasks involved in that role. It also looks at the skills required for the job, compares them with the user's current skills, finds learning resources for missing skills and suggests related career paths.

## What the project does

The user can enter a job role along with an optional job description and current skills.

For example:
Data Analyst
Plumber
Teacher
Graphic Designer
Civil Engineer
Chef
Nurse

The system then performs the following steps:
1. Tries to find the entered job role in O*NET.
2. Gets the relevant occupational and task information when a suitable match is found.
3. Uses an LLM based fallback when a reliable O*NET match is not available.
4. Analyses the tasks associated with the role.
5. Calculates an AI exposure score for each task.
6. Classifies the tasks based on their level of automation and human dependency.
7. Calculates an overall exposure score for the role.
8. Identifies the important skills required for the role.
9. Compares those skills with the user's current skills.
10. Finds missing and partially matched skills.
11. Searches for learning resources based on the role and the missing skill.
12. Suggests related career paths.
13. Displays the complete analysis through a Gradio interface.

## Why task level analysis

The project does not treat a job as something that is either safe from AI or completely replaceable.
Different tasks within the same job can be affected in very different ways.

For example, a task involving repetitive data processing may have high automation potential, while a task that requires physical presence, human judgement or direct interaction with people may remain highly human dependent.

Because of this, the project analyses individual tasks and then combines the results to calculate an overall job exposure score.
The score is only an estimate of task level AI exposure. It is not a prediction that a particular job will disappear.

## O*NET data

O*NET is used as the main occupational data source.
The project uses O*NET files including:
occupation_data.csv
task_statements.csv
task_ratings.csv
software_skills.csv
job_titles.csv

The job title information is used to find a suitable occupational match and the task information is then used for the analysis.
For example, when Plumber is entered, the system can match it to the O*NET occupation Plumbers, Pipefitters, and Steamfitters and use the corresponding task information.

The system also has a fallback path for roles that do not have a reliable O*NET title match. In that case, the LLM generates occupation specific tasks and marks them as estimates.

## RAG

The project uses Retrieval Augmented Generation to provide relevant occupational information to the LLM before it performs the task analysis.
The occupational information is converted into embeddings using Sentence Transformers and stored in a FAISS index.

The embedding model used in the project is:
all-MiniLM-L6-v2

The RAG knowledge base used during development contained 221 documents consisting of occupation information, task information and software or technology information.

The basic flow is:
O*NET data
↓
Text documents
↓
Embeddings
↓
FAISS index
↓
Relevant information retrieved for the role or task
↓
Retrieved information passed to the LLM

This helps ground the LLM's reasoning in occupational data instead of relying only on its general knowledge.

## AI exposure analysis

Each task is analysed using three signals.

## Tool Coverage

This represents how capable current AI or software tools are at performing the task.

## Trend Momentum

This represents how strongly the task is moving towards greater automation.

## Judgment Reliance

This represents how much the task depends on human judgement, accountability, communication, context or physical presence.
Each signal is given a value from 1 to 5.
The LLM provides these signals and Python is used to perform the final calculations.

## Exposure score

The task level AI exposure score is calculated using:
AI Exposure Score = 100 × [0.5 × Tool Coverage + 0.3 × Trend Momentum + 0.2 × (6 − Judgment Reliance)] / 5

Two other values are also used:
Automation Potential = Tool Coverage
Human Dependency = Judgment Reliance

The overall job score is calculated from the task exposure scores. When task importance is available, an importance weighted average is used. Otherwise, equal weighting is used.

## Task classification

Tasks are placed into three categories.

--> Highly Automatable  
Automation Potential >= 4 and Human Dependency <= 2

--> Human-dependent  
Human Dependency >= 4

--> AI-augmented  
Tasks that do not fall into the two categories above.

This classification is intended to show how different parts of a job may be affected differently by AI.

## Skill gap analysis

The project also looks at the skills required for the role.
The required skills are compared with the user's current skills using Sentence Transformer embeddings and cosine similarity.

The current thresholds are:
0.75 and above = Have
0.50 to below 0.75 = Partial
Below 0.50 = Missing

This makes it possible to show not only which skills are important for the job but also which skills the user may need to improve.

## Learning resources

The project includes a dynamic learning resource search.
Instead of giving the same courses for every role, the search uses the job role together with the missing skill.

For example:
Plumber + Pipe Repair + course/training/learning
This allows the recommendations to change depending on the occupation being analysed.

The project also has fallback learning links so that useful search options can still be shown when live search results are not available.

## Career recommendations

The project also suggests related career paths.
The current occupation is represented using its description, tasks and important skills. Embeddings are then used to compare it with other occupation profiles.

The system uses the similar occupations to produce related career recommendations.
These recommendations are intended to show possible adjacent career paths. They are not meant to guarantee that a particular career will be safer from AI.

## Different types of jobs

One of the things I wanted to make sure of was that the system was not limited to software or data related jobs.
The same workflow can be used for technical and non technical occupations.

For example:
Plumber
Teacher
Graphic Designer
Civil Engineer
Chef
Nurse

The skills and learning recommendations change depending on the role.

## Example

One of the tests was done using the role:

Plumber
The system was able to identify the O*NET occupation:
Plumbers, Pipefitters, and Steamfitters
The output included the AI exposure score, task level classifications, required skills, skill gaps, learning resources and related career paths.

This helped verify that the system could work with a non technical occupation and not only with roles such as Data Analyst or Business Intelligence Analyst.

## Gradio application

The final application uses Gradio as the user interface.
The user can enter:
Job Role
Optional Job Description
Optional Current Skills

The application then displays the generated analysis including the exposure score, task analysis, skill gaps, learning resources and career recommendations.

Screenshots of the working application are available in the screenshots folder.

## Technology used

Python
Pandas
NumPy
O*NET
Sentence Transformers
FAISS
Groq hosted LLM
LangChain Groq
Requests
BeautifulSoup
Gradio

## Project structure

AI Job Evolution Analyzer

data
notebook
screenshots
README.md
.gitattributes
requirements.txt
.gitignore

The main implementation is currently in the Jupyter notebook:
notebook/AI_Job_Evolution_Analyzer.ipynb

## How the different parts work together

The LLM is mainly used for reasoning tasks such as task generation, task level exposure analysis, skill synthesis and career explanations.
Python is used for the numerical part of the system such as exposure scoring, classification and other deterministic processing.

RAG is used to retrieve relevant occupational information before the LLM performs its reasoning.
This separation was intentional so that the final numerical calculations are consistent instead of depending on the LLM to directly generate the scores.

## Limitations

This is a project prototype and the results should be treated as estimates.
The quality of some results depends on the LLM, especially when a reliable O*NET occupation match is not available.

Learning resource search also depends on internet access and the quality of the search results.
The exposure score is based on the scoring approach implemented in this project and is not an official industry measure.

## Future improvements

Some things that can be added later are better occupation matching, larger sources of occupational data, stronger validation of the exposure scores, resume based skill extraction, better caching and more detailed learning paths.

## About

This project was built to explore how occupational data, embeddings, retrieval, LLM reasoning and deterministic scoring can be combined into one practical application.

The main idea is to understand which parts of a job are changing because of AI, identify the skills that may need improvement and give the user some possible next steps.