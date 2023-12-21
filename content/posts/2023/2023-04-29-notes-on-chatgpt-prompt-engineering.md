---
title: Notes on ChatGPT Prompt Engineering Course
slug: notes-on-chatgpt-prompt-engineering-course
date: 2023-04-29
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/1/josh-reid-meOFNlRbHmY-unsplash.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/1/josh-reid-meOFNlRbHmY-unsplash.jpg
categories:
- dev
tags:
- chatgpt
- prompt engineering
- openai
---

Andrew Ng's new course ([ChatGPT Prompt Engineering for Developers - DeepLearning.AI](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/)) is a good start to learn about prompt engineering. The following are some notes on the course.

<!--more-->

{{< toc >}}

## Introduction

In the field of natural language processing, prompts are becoming an increasingly popular technique for fine-tuning language models for specific tasks. A prompt is a short piece of text that is fed to a language model as a starting point, after which the model generates a continuation based on its training and programming. 

Prompts can be used to perform a wide range of tasks, including text generation, question-answering, sentiment analysis, and language translation. By providing the model with specific prompts, developers can fine-tune its behavior and improve its performance in specific domains or contexts. The use of prompts has opened up new possibilities for natural language processing and is expected to play a significant role in the development of more advanced language models in the future.


### Why Prompt Engineering

If you want LLMs to write something about Alan Turing, it can be helpful to be clear about whether you want the text to focus on his scientific work or his personal life or his role in history or something else. And it is also better to specify what kind of tone it should take on, such as a professional report or a casual note.


### Two Types of large language models (LLMs)

#### Base LLM

Predicts next word, based on text training data

```text
input: Once upon a time, there was a unicorn  
output: that lived in a magical forest with all her unicorn friends
```

```text
input: What is the capital of France?  
output:  
What is France’s largest city?  
What is France’s population?  
hat is the currency of France?  
```

#### Instruction Tuned LLM

Tries to follow instructions:
- start off with a base LLMs that’s been trained on a huge amount of text data
- Fine-tune on instructions and good attempts at following those instructions.
- further refine using a technique called [RLHF](https://huggingface.co/blog/rlhf) (reinforcement learning from human feedback)
- helpful, honest, harmless, a lot of the practical usage scenarios have been shifting toward instruction tuned LLMs

```text
input: What is the capital of France?  
output: The captial of France is Paris.
```

## Guidelines for Prompting

Using [chat completions endpoint](https://platform.openai.com/docs/guides/chat):

```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())

openai.api_key  = os.getenv('OPENAI_API_KEY')

def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]
```

**Principles**:

- write clear and specific instructions
- give the model time to think

### Principle 1: write clear and specific instructions

#### Tactic 1: Use delimiters to clearly indicate distinct parts of the input
- Triple quotes: `"""`
- Triple backticks: ` ``` `
- Triple dashes: `---`
- Angle brackets: `< >`
- XML tags: `<tag> </tag>`
#### Tactic 2: Ask for a structured output
- JSON
- HTML
#### Tactic 3: Ask the model to check whether conditions are satisfied

```python
text_1 = f"""
Making a cup of tea is easy! First, you need to get some \ 
water boiling. While that's happening, \ 
grab a cup and put a tea bag in it. Once the water is \ 
hot enough, just pour it over the tea bag. \ 
Let it sit for a bit so the tea can steep. After a \ 
few minutes, take out the tea bag. If you \ 
like, you can add some sugar or milk to taste. \ 
And that's it! You've got yourself a delicious \ 
cup of tea to enjoy.
"""
prompt = f"""
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, \ 
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \ 
then simply write \"No steps provided.\"

\"\"\"{text_1}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 1:")
print(response)
```

```text
Completion for Text 1:
Step 1 - Get some water boiling.
Step 2 - Grab a cup and put a tea bag in it.
Step 3 - Once the water is hot enough, pour it over the tea bag.
Step 4 - Let it sit for a bit so the tea can steep.
Step 5 - After a few minutes, take out the tea bag.
Step 6 - Add some sugar or milk to taste.
Step 7 - Enjoy your delicious cup of tea!
```
    
#### Tactic 4: Few-shot prompting

{{< alert info >}}
In natural language processing, [**in-context learning**](https://en.wikipedia.org/wiki/In-context_learning_(natural_language_processing)), **few-shot learning** or **few-shot prompting** is a prompting technique that allows a model to process examples before attempting a task. The method was popularized after the advent of GPT-3 and is considered to be an emergent property of large language models.
{{< /alert >}}

```python
prompt = f"""
Your task is to answer in a consistent style.

<child>: Teach me about patience.

<grandparent>: The river that carves the deepest \ 
valley flows from a modest spring; the \ 
grandest symphony originates from a single note; \ 
the most intricate tapestry begins with a solitary thread.

<child>: Teach me about resilience.
"""
response = get_completion(prompt)
print(response)
```

```<grandparent>: Resilience is like a tree that bends with the wind but never breaks. It is the ability to bounce back from adversity and keep moving forward, even when things get tough. Just like a tree that grows stronger with each storm it weathers, resilience is a quality that can be developed and strengthened over time.```


### Principle 2: Give the model time to “think”

#### Tactic 1: Specify the steps required to complete a task

```python
prompt_2 = f"""
Your task is to perform the following actions: 
1 - Summarize the following text delimited by 
  <> with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the 
  following keys: french_summary, num_names.

Use the following format:
Text: <text to summarize>
Summary: <summary>
Translation: <summary translation>
Names: <list of names in Italian summary>
Output JSON: <json with summary and num_names>

Text: <{text}>
"""
response = get_completion(prompt_2)
print("\nCompletion for prompt 2:")
print(response)
```

```text
Completion for prompt 2:
Summary: Jack and Jill go on a quest to fetch water, but misfortune strikes and they tumble down the hill, returning home slightly battered but with their adventurous spirits undimmed. 
Translation: Jack et Jill partent en quête d'eau, mais la malchance frappe et ils dégringolent la colline, rentrant chez eux légèrement meurtris mais avec leurs esprits aventureux intacts.
Names: Jack, Jill
Output JSON: {"french_summary": "Jack et Jill partent en quête d'eau, mais la malchance frappe et ils dégringolent la colline, rentrant chez eux légèrement meurtris mais avec leurs esprits aventureux intacts.", "num_names": 2}
```

#### Tactic 2: Instruct the model to work out its own solution before rushing to a conclusion

````python
prompt = f"""
Your task is to determine if the student's solution \
is correct or not.
To solve the problem do the following:
- First, work out your own solution to the problem. 
- Then compare your solution to the student's solution \ 
and evaluate if the student's solution is correct or not. 
Don't decide if the student's solution is correct until 
you have done the problem yourself.

Use the following format:
Question:
```
question here
```
Student's solution:
```
student's solution here
```
Actual solution:
```
steps to work out the solution and your solution here
```
Is the student's solution the same as actual solution \
just calculated:
```
yes or no
```
Student grade:
```
correct or incorrect
```

Question:
```
I'm building a solar power installation and I need help \
working out the financials. 
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations \
as a function of the number of square feet.
``` 
Student's solution:
```
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
```
Actual solution:
"""
response = get_completion(prompt)
print(response)
````

```text
Let x be the size of the installation in square feet.

Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 10x

Total cost: 100x + 250x + 100,000 + 10x = 360x + 100,000

Is the student's solution the same as actual solution just calculated:
No

Student grade:
Incorrect
```


### Model Limitations: Hallucinations

Reducing hallucinations:

- first find relevant information
- then answer the question based on the relevant information

## Iterative Prompt Development

> Idea ⇒ Implementation (code/data/prompt) ⇒ Experimental result (Error Analysis) ⇒ Idea …

### Prompt guidelines by iterative process

- Try somthing, be clear and specific
- Analyze why result does not give desired output
- Clarify instructions, give more time to think
- Refine the idea and the prompt with a batch of examples
- Repeat

### Example

```python
fact_sheet_chair = """
OVERVIEW
- Part of a beautiful family of mid-century inspired office furniture, 
including filing cabinets, desks, bookcases, meeting tables, and more.
- Several options of shell color and base finishes.
- Available with plastic back and front upholstery (SWC-100) 
or full upholstery (SWC-110) in 10 fabric and 6 leather options.
- Base finish options are: stainless steel, matte black, 
gloss white, or chrome.
- Chair is available with or without armrests.
- Suitable for home or business settings.
- Qualified for contract use.

CONSTRUCTION
- 5-wheel plastic coated aluminum base.
- Pneumatic chair adjust for easy raise/lower action.

DIMENSIONS
- WIDTH 53 CM | 20.87”
- DEPTH 51 CM | 20.08”
- HEIGHT 80 CM | 31.50”
- SEAT HEIGHT 44 CM | 17.32”
- SEAT DEPTH 41 CM | 16.14”

OPTIONS
- Soft or hard-floor caster options.
- Two choices of seat foam densities: 
 medium (1.8 lb/ft3) or high (2.8 lb/ft3)
- Armless or 8 position PU armrests 

MATERIALS
SHELL BASE GLIDER
- Cast Aluminum with modified nylon PA6/PA66 coating.
- Shell thickness: 10 mm.
SEAT
- HD36 foam

COUNTRY OF ORIGIN
- Italy
"""
```

```python
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

At the end of the description, include every 7-character 
Product ID in the technical specification.

After the description, include a table that gives the 
product's dimensions. The table should have two columns.
In the first column include the name of the dimension. 
In the second column include the measurements in inches only.

Give the table the title 'Product Dimensions'.

Format everything as HTML that can be used in a website. 
Place the description in a <div> element.

Use at most 50 words.

Technical specifications: ```{fact_sheet_chair}```
"""

response = get_completion(prompt)
print(response)
```

```python
from IPython.display import display, HTML
display(HTML(response))
```

## Summarizing

### Example: Summarizing user reviews for brevity

#### summarize with a word/sentence/character limit

```python
prompt = f"""
Your task is to generate a short summary of a product \
review from an ecommerce site. 

Summarize the review below, delimited by triple 
backticks, in at most 30 words. 

Review: ```{prod_review}```
"""
```

#### summarize with a focus on shipping and delivery

```python
prompt = f"""
Your task is to generate a short summary of a product \
review from an ecommerce site to give feedback to the \
Shipping deparmtment. 

Summarize the review below, delimited by triple 
backticks, in at most 30 words, and focusing on any aspects \
that mention shipping and delivery of the product. 

Review: ```{prod_review}```
"""
```

#### summarize with a focus on price and value

```python
prompt = f"""
Your task is to generate a short summary of a product \
review from an ecommerce site to give feedback to the \
pricing deparmtment, responsible for determining the \
price of the product.  

Summarize the review below, delimited by triple 
backticks, in at most 30 words, and focusing on any aspects \
that are relevant to the price and perceived value. 

Review: ```{prod_review}```
"""
```


## Inferring

**Inferring** allows for the quick building and implementation of natural language processing systems for both experienced and novice machine learning developers, significantly reducing the time and effort required for complex tasks.

### Sentiment classification (positive/negative)

```python
lamp_review = """
Needed a nice lamp for my bedroom, and this one had \
additional storage and not too high of a price point. \
Got it fast.  The string to our lamp broke during the \
transit and the company happily sent over a new one. \
Came within a few days as well. It was easy to put \
together.  I had a missing part, so I contacted their \
support and they very quickly got me the missing piece! \
Lumina seems to me to be a great company that cares \
about their customers and products!!
"""
```

```python
prompt = f"""
What is the sentiment of the following product review, 
which is delimited with triple backticks?

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

```python
prompt = f"""
What is the sentiment of the following product review, 
which is delimited with triple backticks?

Give your answer as a single word, either "positive" \
or "negative".

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

#### Identify types of emotions

```python
prompt = f"""
Identify a list of emotions that the writer of the \
following review is expressing. Include no more than \
five items in the list. Format your answer as a list of \
lower-case words separated by commas.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

```python
happy, satisfied, grateful, impressed, content
```

#### Identify anger

```python
prompt = f"""
Is the writer of the following review expressing anger?\
The review is delimited with triple backticks. \
Give your answer as either yes or no.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

#### Extract product and company name from customer reviews

```python
prompt = f"""
Identify the following items from the review text: 
- Item purchased by reviewer
- Company that made the item

The review is delimited with triple backticks. \
Format your response as a JSON object with \
"Item" and "Brand" as the keys. 
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
  
Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

```python
{
  "Item": "lamp",
  "Brand": "Lumina"
}
```
    
#### Doing multiple tasks at once

```python
prompt = f"""
Identify the following items from the review text: 
- Sentiment (positive or negative)
- Is the reviewer expressing anger? (true or false)
- Item purchased by reviewer
- Company that made the item

The review is delimited with triple backticks. \
Format your response as a JSON object with \
"Sentiment", "Anger", "Item" and "Brand" as the keys.
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
Format the Anger value as a boolean.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

```python
{
  "Sentiment": "positive",
  "Anger": false,
  "Item": "lamp with additional storage",
  "Brand": "Lumina"
}
```

### Topic extraction

```python
story = """
In a recent survey conducted by the government, 
public sector employees were asked to rate their level 
of satisfaction with the department they work at. 
The results revealed that NASA was the most popular 
department with a satisfaction rating of 95%.

One NASA employee, John Smith, commented on the findings, 
stating, "I'm not surprised that NASA came out on top. 
It's a great place to work with amazing people and 
incredible opportunities. I'm proud to be a part of 
such an innovative organization."

The results were also welcomed by NASA's management team, 
with Director Tom Johnson stating, "We are thrilled to 
hear that our employees are satisfied with their work at NASA. 
We have a talented and dedicated team who work tirelessly 
to achieve our goals, and it's fantastic to see that their 
hard work is paying off."

The survey also revealed that the 
Social Security Administration had the lowest satisfaction 
rating, with only 45% of employees indicating they were 
satisfied with their job. The government has pledged to 
address the concerns raised by employees in the survey and 
work towards improving job satisfaction across all departments.
"""
```

```python
prompt = f"""
Determine five topics that are being discussed in the \
following text, which is delimited by triple backticks.

Make each item one or two words long. 

Format your response as a list of items separated by commas.

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
```

```python
government survey, job satisfaction, NASA, Social Security Administration, employee concerns
```

#### Infer 5 topics

```python
prompt = f"""
Determine five topics that are being discussed in the \
following text, which is delimited by triple backticks.

Make each item one or two words long. 

Format your response as a list of items separated by commas.

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
response.split(sep=',')
```

```python
['government survey',
  ' job satisfaction',
  ' NASA',
  ' Social Security Administration',
  ' employee concerns']
```

#### Make a news alert for certain topics (zero shot)

```python
topic_list = [
    "nasa", "local government", "engineering", 
    "employee satisfaction", "federal government"
]
```

```python
prompt = f"""
Determine whether each item in the following list of \
topics is a topic in the text below, which
is delimited with triple backticks.

Give your answer as list with 0 or 1 for each topic.\

List of topics: {", ".join(topic_list)}

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
```

```
nasa: 1
local government: 0
engineering: 0
employee satisfaction: 1
federal government: 1
```

```python
topic_dict = {i.split(': ')[0]: int(i.split(': ')[1]) for i in response.split(sep='\n')}
if topic_dict['nasa'] == 1:
    print("ALERT: New NASA story!")
```


## Transforming

Large Language Models (LLMs) can perform various text transformation tasks such as 

- language translation,
- spelling and grammar checking,
- tone adjustment, and
- format conversion.

### Translation

```python
prompt = f"""
Translate the following text to Spanish in both the \
formal and informal forms: 
'Would you like to order a pillow?'
"""
response = get_completion(prompt)
print(response)
```

#### Universal Translator

```python
user_messages = [
  "La performance du système est plus lente que d'habitude.",  # System performance is slower than normal         
  "Mi monitor tiene píxeles que no se iluminan.",              # My monitor has pixels that are not lighting
  "Il mio mouse non funziona",                                 # My mouse is not working
  "Mój klawisz Ctrl jest zepsuty",                             # My keyboard has a broken control key
  "我的屏幕在闪烁"                                               # My screen is flashing
]
```

```python
for issue in user_messages:
    prompt = f"Tell me what language this is: ```{issue}```"
    lang = get_completion(prompt)
    print(f"Original message ({lang}): {issue}")

    prompt = f"""
    Translate the following  text to English \
    and Korean: ```{issue}```
    """
    response = get_completion(prompt)
    print(response, "\n")
```

```
Original message (This is French.): La performance du système est plus lente que d'habitude.
English: The system performance is slower than usual.
Korean: 시스템 성능이 평소보다 느립니다.

Original message (This is Spanish.): Mi monitor tiene píxeles que no se iluminan.
English: My monitor has pixels that don't light up.
Korean: 내 모니터에는 불이 켜지지 않는 픽셀이 있습니다.

Original message (This is Italian.): Il mio mouse non funziona
English: My mouse is not working.
Korean: 내 마우스가 작동하지 않습니다.

Original message (This is Polish.): Mój klawisz Ctrl jest zepsuty
English: My Ctrl key is broken.
Korean: 제 Ctrl 키가 고장 났어요.

Original message (This is Chinese (Simplified).): 我的屏幕在闪烁
English: My screen is flickering.
Korean: 내 화면이 깜빡입니다.
```


### Tone Transformation

Writing can vary based on the intended audience. ChatGPT can produce different tones.

```python
prompt = f"""
Translate the following from slang to a business letter: 
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```

{{< alert success >}}
Dear Sir/Madam,

I am writing to bring to your attention a standing lamp that I believe may be of interest to you. Please find attached the specifications for your review.

Thank you for your time and consideration.

Sincerely,

Joe
{{< /alert >}}

### Format Conversion

should describe the input and output formats

```python
data_json = { "resturant employees" :[ 
    {"name":"Shyam", "email":"shyamjaiswal@gmail.com"},
    {"name":"Bob", "email":"bob32@gmail.com"},
    {"name":"Jai", "email":"jai87@gmail.com"}
]}

prompt = f"""
Translate the following python dictionary from JSON to an HTML \
table with column headers and title: {data_json}
"""
response = get_completion(prompt)
print(response)
```

```python
from IPython.display import display, Markdown, Latex, HTML, JSON
display(HTML(response))
```

### Spellcheck/Grammar check

Use LLM to help proofread.

```python
text = [ 
  "The girl with the black and white puppies have a ball.",  # The girl has a ball.
  "Yolanda has her notebook.", # ok
  "Its going to be a long day. Does the car need it’s oil changed?",  # Homonyms
  "Their goes my freedom. There going to bring they’re suitcases.",  # Homonyms
  "Your going to need you’re notebook.",  # Homonyms
  "That medicine effects my ability to sleep. Have you heard of the butterfly affect?", # Homonyms
  "This phrase is to cherck chatGPT for speling abilitty"  # spelling
]
for t in text:
    prompt = f"""Proofread and correct the following text
    and rewrite the corrected version. If you don't find
    and errors, just say "No errors found". Don't use 
    any punctuation around the text:
    ```{t}```"""
    response = get_completion(prompt)
    print(response)
```

```python
text = f"""
Got this for my daughter for her birthday cuz she keeps taking \
mine from my room.  Yes, adults also like pandas too.  She takes \
it everywhere with her, and it's super soft and cute.  One of the \
ears is a bit lower than the other, and I don't think that was \
designed to be asymmetrical. It's a bit small for what I paid for it \
though. I think there might be other options that are bigger for \
the same price.  It arrived a day earlier than expected, so I got \
to play with it myself before I gave it to my daughter.
"""
prompt = f"proofread and correct this review: ```{text}```"
response = get_completion(prompt)
print(response)
```

```python
from redlines import Redlines

diff = Redlines(text,response)
display(Markdown(diff.output_markdown))
```

{{< alert success >}}
**~~Got~~ I got** this for my **~~daughter for her~~ daughter's** birthday **~~cuz~~ because** she keeps taking mine from my **~~room.~~ room.** Yes, adults also like pandas **~~too.~~ too.** She takes it everywhere with her, and it's super soft and **~~cute. One~~ cute. However, one** of the ears is a bit lower than the other, and I don't think that was designed to be asymmetrical. **~~It's~~ Additionally, it's** a bit small for what I paid for **~~it though.~~ it.** I think there might be other options that are bigger for the same **~~price. It~~ price. On the positive side, it** arrived a day earlier than expected, so I got to play with it myself before I gave it to my **~~daughter.~~ daughter.**
{{< /alert >}}

#### Rewrite with specific style

{{< alert info >}}
[**APA style**](https://en.wikipedia.org/wiki/APA_style) is a writing format and citation style developed by the American Psychological Association. It is commonly used in social sciences, education, and other fields to format research papers, essays, and other written works. The APA style includes guidelines for formatting, citing sources, and creating references.
{{< /alert >}}

```python
prompt = f"""
proofread and correct this review. Make it more compelling. 
Ensure it follows APA style guide and targets an advanced reader. 
Output in markdown format.
Text: ```{text}```
"""
response = get_completion(prompt)
display(Markdown(response))
```

{{< alert success >}}
Title: A Soft and Cute Panda Plush Toy for All Ages

Introduction: As a parent, finding the perfect gift for your child's birthday can be a daunting task. However, I stumbled upon a soft and cute panda plush toy that not only made my daughter happy but also brought joy to me as an adult. In this review, I will share my experience with this product and provide an honest assessment of its features.

Product Description: The panda plush toy is made of high-quality materials that make it super soft and cuddly. Its cute design is perfect for children and adults alike, making it a versatile gift option. The toy is small enough to carry around, making it an ideal companion for your child on their adventures.

Pros: The panda plush toy is incredibly soft and cute, making it an excellent gift for children and adults. Its small size makes it easy to carry around, and its design is perfect for snuggling. The toy arrived a day earlier than expected, which was a pleasant surprise.

Cons: One of the ears is a bit lower than the other, which makes the toy asymmetrical. Additionally, the toy is a bit small for its price, and there might be other options that are bigger for the same price.

Conclusion: Overall, the panda plush toy is an excellent gift option for children and adults who love cute and cuddly toys. Despite its small size and asymmetrical design, the toy's softness and cuteness make up for its shortcomings. I highly recommend this product to anyone looking for a versatile and adorable gift option.

{{< /alert >}}
    

## Expanding

```python
def get_completion(prompt, model="gpt-3.5-turbo",temperature=0): # Andrew mentioned that the prompt/ completion paradigm is preferable for this class
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]
```

### Example: automatically writing emails

#### Generate customer service emails that are tailored to each customer's review.

```python
# given the sentiment from the lesson on "inferring",
# and the original customer message, customize the email
sentiment = "negative"

# review for a blender
review = f"""
So, they still had the 17 piece system on seasonal \
sale for around $49 in the month of November, about \
half off, but for some reason (call it price gouging) \
around the second week of December the prices all went \
up to about anywhere from between $70-$89 for the same \
system. And the 11 piece system went up around $10 or \
so in price also from the earlier sale price of $29. \
So it looks okay, but if you look at the base, the part \
where the blade locks into place doesn’t look as good \
as in previous editions from a few years ago, but I \
plan to be very gentle with it (example, I crush \
very hard items like beans, ice, rice, etc. in the \ 
blender first then pulverize them in the serving size \
I want in the blender then switch to the whipping \
blade for a finer flour, and use the cross cutting blade \
first when making smoothies, then use the flat blade \
if I need them finer/less pulpy). Special tip when making \
smoothies, finely cut and freeze the fruits and \
vegetables (if using spinach-lightly stew soften the \ 
spinach then freeze until ready for use-and if making \
sorbet, use a small to medium sized food processor) \ 
that you plan to use that way you can avoid adding so \
much ice if at all-when making your smoothie. \
After about a year, the motor was making a funny noise. \
I called customer service but the warranty expired \
already, so I had to buy another one. FYI: The overall \
quality has gone done in these types of products, so \
they are kind of counting on brand recognition and \
consumer loyalty to maintain sales. Got it in about \
two days.
"""
```

```python
prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.
Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.
If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
Customer review: ```{review}```
Review sentiment: {sentiment}
"""
response = get_completion(prompt)
print(response)
```

{{< alert success >}}
Dear Valued Customer,

Thank you for taking the time to leave a review about our product. We are sorry to hear that you experienced an increase in price and that the quality of the product did not meet your expectations. We apologize for any inconvenience this may have caused you.

We would like to assure you that we take all feedback seriously and we will be sure to pass your comments along to our team. If you have any further concerns, please do not hesitate to reach out to our customer service team for assistance.

Thank you again for your review and for choosing our product. We hope to have the opportunity to serve you better in the future.

Best regards,

AI customer agent
{{< /alert >}}

### Temperature

{{< alert info >}}
The **temperature** parameter is used in the generation of text using language models to control the level of **randomness** and **creativity** in the generated text. It controls the degree of randomness in the output by scaling the logits (**uncertainty estimates**) produced by the model before applying the softmax function to obtain the probability distribution over the vocabulary. 

A higher temperature leads to a more diverse and creative output, while a lower temperature leads to a more conservative and predictable output.
{{< /alert >}}


#### Remind the model to use details from the customer’s email

```python
prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.
Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.
If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
Customer review: ```{review}```
Review sentiment: {sentiment}
"""
response = get_completion(prompt, temperature=0.7)
print(response)
```

{{< alert success >}}
Dear valued customer,

Thank you for taking the time to write a review about our product. We are sorry to hear that the prices of our 17 piece system went up in December and that you experienced issues with the motor of your previous unit. We apologize for any inconvenience this may have caused you.

If you have any further concerns or questions, please do not hesitate to reach out to our customer service team. They will be more than happy to assist you in any way possible.

Thank you again for your feedback, and we hope that you will continue to enjoy our products.

Best regards,

AI customer agent
{{< /alert >}}
    

## Chatbot

### Setup

```python
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]

def get_completion_from_messages(messages, model="gpt-3.5-turbo", temperature=0):
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
#     print(str(response.choices[0].message))
    return response.choices[0].message["content"]
```

{{< alert info >}}
In the context of a conversational system or chatbot:

1. **System**: The system refers to the chatbot or conversational agent that is designed to interact with users and respond to their input. The system's role is to interpret user input and generate an appropriate response based on its programming and training.
2. **User**: The user refers to the person or people interacting with the chatbot or conversational agent. The user's role is to input text or speech to the system and receive responses.
3. **Assistant**: The assistant refers to the chatbot or conversational agent when it is designed to provide helpful information or assistance to the user. The assistant's role is to provide guidance, answer questions, or perform tasks on behalf of the user, often in a specific domain or context.
{{< /alert >}}

```python
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},
{'role':'user', 'content':'Hi, my name is Isa'},
{'role':'assistant', 'content': "Hi Isa! It's nice to meet you. \
Is there anything I can help you with today?"},
{'role':'user', 'content':'Yes, you can remind me, What is my name?'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

```text
Your name is Isa.
```

### OrderBot

```python
def collect_messages(_):
    prompt = inp.value_input
    inp.value = ''
    context.append({'role':'user', 'content':f"{prompt}"})
    response = get_completion_from_messages(context) 
    context.append({'role':'assistant', 'content':f"{response}"})
    panels.append(
        pn.Row('User:', pn.pane.Markdown(prompt, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600, style={'background-color': '#F6F6F6'})))
 
    return pn.Column(*panels)
```

```python
import panel as pn  # GUI
pn.extension()

panels = [] # collect display 

context = [ {'role':'system', 'content':"""
You are OrderBot, an automated service to collect orders for a pizza restaurant. \
You first greet the customer, then collects the order, \
and then asks if it's a pickup or delivery. \
You wait to collect the entire order, then summarize it and check for a final \
time if the customer wants to add anything else. \
If it's a delivery, you ask for an address. \
Finally you collect the payment.\
Make sure to clarify all options, extras and sizes to uniquely \
identify the item from the menu.\
You respond in a short, very conversational friendly style. \
The menu includes \
pepperoni pizza  12.95, 10.00, 7.00 \
cheese pizza   10.95, 9.25, 6.50 \
eggplant pizza   11.95, 9.75, 6.75 \
fries 4.50, 3.50 \
greek salad 7.25 \
Toppings: \
extra cheese 2.00, \
mushrooms 1.50 \
sausage 3.00 \
canadian bacon 3.50 \
AI sauce 1.50 \
peppers 1.00 \
Drinks: \
coke 3.00, 2.00, 1.00 \
sprite 3.00, 2.00, 1.00 \
bottled water 5.00 \
"""} ]  # accumulate messages

inp = pn.widgets.TextInput(value="Hi", placeholder='Enter text here…')
button_conversation = pn.widgets.Button(name="Chat!")

interactive_conversation = pn.bind(collect_messages, button_conversation)

dashboard = pn.Column(
    inp,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True, height=300),
)

dashboard
```

> **Assistant**: Hello! Welcome to our pizza restaurant. What can I get for you today?
> 
> 
> **User**: **what about price**
> 
> **Assistant**:
> 
> Sure, I can provide you with the prices. Here is our menu:
> 
> - Pepperoni pizza: 12.95 (small), 10.00 (medium), 7.00 (large)
> - Cheese pizza: 10.95 (small), 9.25 (medium), 6.50 (large)
> - Eggplant pizza: 11.95 (small), 9.75 (medium), 6.75 (large)
> - Fries: 4.50 (small), 3.50 (large)
> - Greek salad: 7.25
> - Toppings: extra cheese 2.00, mushrooms 1.50, sausage 3.00, canadian bacon 3.50, AI sauce 1.50, peppers 1.00
> - Drinks: coke 3.00 (small), 2.00 (medium), 1.00 (large), sprite 3.00 (small), 2.00 (medium), 1.00 (large), bottled water 5.00
> 
> Let me know if you have any questions or if you’re ready to place an order.
> 
> **User**: **are you kidding me ? small are much more expensive that large ones !**
> 
> **Assistant**:
> 
> I apologize for the confusion. I made a mistake in my previous message. Here are the correct prices:
> 
> - Pepperoni pizza: 7.00 (small), 10.00 (medium), 12.95 (large)
> - Cheese pizza: 6.50 (small), 9.25 (medium), 10.95 (large)
> - Eggplant pizza: 6.75 (small), 9.75 (medium), 11.95 (large)
> - Fries: 3.50 (small), 4.50 (large)
> - Greek salad: 7.25
> - Toppings: extra cheese 2.00, mushrooms 1.50, sausage 3.00, canadian bacon 3.50, AI sauce 1.50, peppers 1.00
> - Drinks: coke 1.00 (small), 2.00 (medium), 3.00 (large), sprite 1.00 (small), 2.00 (medium), 3.00 (large), bottled water 5.00
> 
> I hope this clears things up. Let me know if you have any other questions or if you’re ready to place an order.
> 
> **User**: it’s ok, thanks for the update. I would like medium cheese pizza and small coke for drink
> 
> **Assistant**: Great! So that’s a medium cheese pizza for 9.25 and a small coke for 1.00. Is there anything else you’d like to add to your order?
> 
> **User**: no, thanks
> 
> **Assistant**: Alright, just to confirm, you’d like to order a medium cheese pizza for 9.25 and a small coke for 1.00. Is that correct?
> 
> **User**: yes
> 
> **Assistant**: Got it! Will that be for pickup or delivery?
> 
> **User**: pickup is ok
> 
> **Assistant**: Perfect! Your order for a medium cheese pizza and a small coke for pickup has been placed. Your total comes to 10.25. Is there anything else I can help you with?
> 

```python
messages =  context.copy()
messages.append(
{'role':'system', 'content':'create a json summary of the previous food order. Itemize the price for each item\
 The fields should be 1) pizza, include size 2) list of toppings 3) list of drinks, include size   4) list of sides include size  5)total price '},    
)
 #The fields should be 1) pizza, price 2) list of toppings 3) list of drinks, include size include price  4) list of sides include size include price, 5)total price '},    

response = get_completion_from_messages(messages, temperature=0)
print(response)
```

````
Sure, here's a JSON summary of your order:

```
{
  "pizza": {
    "type": "cheese",
    "size": "medium",
    "price": 9.25
  },
  "toppings": [],
  "drinks": [
    {
      "type": "coke",
      "size": "small",
      "price": 1.00
    }
  ],
  "sides": [],
  "total_price": 10.25
}
```

Let me know if you have any questions or if you'd like to make any changes to your order.
````

## Summary

- Principles:
  - Write clear and specific instructions
  - Give the model time to "think"
- Iterative prompt development
- Capabilities: 
  - Summarizing
  - Inferring
  - Transforming
  - Expanding
- Building a chatbot