# RAG_challenge_v2

RAG challenge sourse data description:
 - subset.json - список компаний и какие данные о них предварительно известны (согласно переписке в Discord - не все данные верные, они получены быстрым поиском более слабыми моделям, можно полагаться только на имена и общее кол-во компаний).
 - questions.json - список вопросов, на которые RAG должен уметь отвечать (types: number, boolean, name/names).
 - reports - pdf files can be finded here: https://rag.timetoact.at/data/r2.0/pdfs.zip
 - test_report.pdf  - example of report.

## Main strategy of decision

### 1. Questions explorartion - done:
- Study the list of questions and reconsider the types of questions.
- Evaluate the percentage contribution of each question type to the overall sample. Understand where it is most beneficial to focus first.

> Out of the 100 questions,  
> 24 are boolean‐type (which can be answered using the boolean flags in subset.json), while the remaining  
> 58 (number-type) plus  
> 18 (name/qualitative type) 
> questions—totalling 76 questions—require specific numerical or descriptive data that are not included in subset.json.
- Determine what data about companies and what types of data need to be extracted from the reports to answer all the questions. (Full list is collected in data_gap_types.md). Let's call them "measures".
- Create standard data models for company information. JSON schema or Pydantic classes. (Prepaired in pdf_preparation.ipynb)

### 2. PDF preparation - partly done: 
- Split the documents into PDF pages.
- Create a markdowned version of the reports text, with metadata like the page number of the report, which we then ask the LLM to simply substitute into the response.

Here we can also allocate a separate module that does pre-cleaning and PDF upload to vector storage with content containing one pdf page and do retrieving search for every question.
So we can feed the LLM a large bounded text with markup and page number - this avoids the LLM having to do page counting. Which they are not very good at.

### 3. Data extraction and knowledge map preparation - partly done: 
- Prepare prompts for extracting each type of data:
  - Numerical monetary (as they may be in different currencies = units of measurement),
  - Other numerical operational,
  - Qualitative,
  - Boolean.
- For reports that fit completely in the model contextual window:
   - Each prepaired report should be loaded into the LLM one by one, and ask the model to find all the data from the data model (JSON schema or Pydantic classes, in code Pydantic classes) in model should be included the number of page.
- Reports that do not fit completely in the model contextual window:
  - v.1. Break the report into chunks of maximum window size and send each of them separately to the LLM with a request to find all possible metrics/"measures". Then collect all found metrics/"measures" in each of the chunks and assemble them into one.
  - v.2. Each prepaired report should be broken down into semantic sections, sized to fit the context window. For this purpose, each page of the report is submitted separately to the LLM with a request to mark which data from the list are present on it - please output the list. In this way we get the marked dataset and reorganize the report so that only relevant pages are submitted to the LLM for searching each metric/"measure" (the sequence of pages must be respected). Here we may encounter that all the pages mentioning any of the indicators will be larger than the context. As part of the solution to this problem, we can create a graph database and search this database to gather context for each metrics/"measure".

The prompt for numerical monetary data has been tested and provided. Structured output was used with the OpenAI GPT model="o3-mini-2025-01-31".
Results - monetary_data_extraction_for_test_report.json

- In this way we collect a knowledge map for the provided business reports.

### 4. Question answering - need to do: 
- For each question in questions.json, provide the prepared knowledge map as the context for generating the answer.





