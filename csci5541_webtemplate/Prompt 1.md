[ChatGPT conversation](https://chatgpt.com/share/690d31b7-87cc-8004-89d3-3d88a58848b5)

Instructions:

```txt
1.5 Midterm office hour participation

Requirement. Your team must schedule an office hour meeting with your assigned mentor (15 to 20
minutes) before the due date and discuss your intermediate results and progress. You have to create
a project webpage (template: https://github.com/minnesotanlp/csci5541-project-template) and show
them during the discussion with your mentor.
The mentor expects you to give an update on your progress results, ask questions (blockers, technical
questions), and consult with your plan until the final presentation. After the meeting, you have to
report what intermediate progress you made, summarize the feedback you receive from your mentor
and explain how you plan to address it. Your response must be uploaded directly to Canvas.
Evaluation. Below is a rubric for midterm evaluation:
Rubric (5 points) for Midterm Office Hour Participation:
(1 point) Additional development of your ideas after the proposal
(1 point) Submission of the project webpage
(2 points) Preliminary results and comparison to the baseline performance
,→ (e.g., experimental results, findings, visualization)
(1 point) Plan to address the mentor's feedback and plan until the end of the
,→ semester

```

Information about our project:
```md
Problem Definition: Companies need a way to extract valuable and appropriate info from business emails, Slack, etc., while discarding irrelevant, inappropriate (private, personal), and unreliable (sarcastic) info.

Method: Modular based Colab pipeline:
Receive ‘informal’ information (gmail API for demo)
Preprocess message into individual claims
Score each claim via MoE classifier modules (fine tuned RoBERTa-large models - trained on Enron emails, and potentially also RLAIF).
Chat terminal for RAG-QA testing using ‘Claim DB’.

Data: 1700 Enron emails - labeled with type (personal, corporate, etc) and emotional tone (neutral, sarcasm, etc).

Novelty: Prior research focus on claim creation, DB structure (i.e. graphRAG) or RRF. Our project focuses on reducing RAG ‘rubbish in, rubbish out’ by classifying and removing non-appropriate information at source.

Forward Plan:
Build vanilla RAG w/ vector database
Fine tune custom-LMs for semantic scoring (sarcasm, intent, etc.)
Evaluate the efficacy of custom-LM models ranking vs. GPT5 ranking.
Evaluate the efficacy of retrieval from pre-filtered data vs. vanilla RAG.
[If time: Compare to semantic RRF ranking (i.e. post-filter) both for Vanilla RAG and Pre-filtered RAG]
Some questions we expect to explore a bit:
What are some sentiments that are important to capture from source material?
Where is the line for data privacy when scraping company emails?
At what point should a source be discarded for confidentiality (i.e. PII)?
```


Template html from the given link (I have updated a few lines to our project's name)

```html
<!DOCTYPE html>
<!-- saved from url=(0014)about:internet -->
<html lang=" en-US"><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>NLP Class Project | Fall 2025 CSCI 5541 | University of Minnesota</title>

  <link rel="stylesheet" href="./files/bulma.min.css" />

  <link rel="stylesheet" href="./files/styles.css">
  <link rel="preconnect" href="https://fonts.gstatic.com/">
  <link href="./files/css2" rel="stylesheet">
  <link href="./files/css" rel="stylesheet">


  <base href="." target="_blank"></head>


<body>
  <div>
    <div class="wrapper">
      <h1 style="font-family: &#39;Lato&#39;, sans-serif;">Appropriateness Filtering for Corporate RAG Database Storage</h1>
      <h4 style="font-family: &#39;Lato&#39;, sans-serif; ">Fall 2025 CSCI 5541 NLP: Class Project - University of Minnesota</h4>
      <h4 style="font-family: &#39;Lato&#39;, sans-serif; ">Netwatch</h4>

      <div class="authors-wrapper">
        
        <div class="author-container">
          <div class="author-image">
                        
              <img src="">
            
            
          </div>
          <p>
                        
              Member 1
            
          </p>
        </div>
        
        <div class="author-container">
          <div class="author-image">
            
            <img src="">
            
          </div>
          <p>
            
            Member 2
            
          </p>
        </div>
        
        <div class="author-container">
          <div class="author-image">
            
              <img src="">            
            
          </div>
          <p>
              Member 3
          </p>
        </div>
        
        <div class="author-container">
          <div class="author-image">
                        
              <img src="">
            
          </div>
          <p>
            Member 4
          </p>
        </div>
        
      </div>

      <br/>

      <div class="authors-wrapper">
        <div class="publication-links">
          <!-- Github link -->
          <span class="link-block">
            <a
              href=""
              target="_blank"
              class="external-link button is-normal is-rounded is-dark is-outlined"
            >
            <span>Final Report</span>
            </a>
          </span>
          <span class="link-block">
            <a
              href=""
              target="_blank"
              class="external-link button is-normal is-rounded is-dark is-outlined"
            >
            <span>Code</span>
            </a>
          </span>      
          <span class="link-block">
            <a
              href=""
              target="_blank"
              class="external-link button is-normal is-rounded is-dark is-outlined"
            >
            <span>Model Weights</span>
            </a>
          </span>              
        </div>
      </div>


    </div>
  </div>





  
  


  <div class="wrapper">
    <hr>
    
    <h2 id="abstract">Abstract</h2>

<p>One or two sentences on the motivation behind the problem you are solving. One or two sentences describing the approach you took. One or two sentences on the main result you obtained.</p>

<hr>

<h2 id="teaser">Teaser Figure</h2>

<p>TODO: I have updated this image path but now the description is irrelevant. Template continues: A figure that conveys the main idea behind the project or the main application being addressed. This figure is from <a href="https://arxiv.org/pdf/2210.07469">StyLEx</a>.</p>

<p class="sys-img"><img src="./files/prj diagram.png" alt="imgname"></p>


<h3 id="the-timeline-and-the-highlights">Any subsection</h3>

<p>If you need to explain more about your figure</p>

<hr>

<h2 id="introduction">Introduction / Background / Motivation</h2>

<p>
<b>What did you try to do? What problem did you try to solve? Articulate your objectives using absolutely no jargon.</b>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
</p>

<p>
<b>How is it done today, and what are the limits of current practice?</b>
</p>
<p>
Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
<p>

<p>
<b>Who cares? If you are successful, what difference will it make?</b>
</p>
<p>
Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>

<hr>

<h2 id="approach">Approach</h2>

<p>
<b>What did you do exactly? How did you solve the problem? Why did you think it would be successful? Is anything new in your approach?</b>
</p>

<p>
Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
</p>

<p>
<b>What problems did you anticipate? What problems did you encounter? Did the very first thing you tried work?</b>
</p>

<p>
Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo.
</p>

<hr>
    
<h2 id="results">Results</h2>
<p>
<b>How did you measure success? What experiments were used? What were the results, both quantitative and qualitative? Did you succeed? Did you fail? Why?</b>
</p>
<p>
Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt.
</p>
<table>
  <thead>
    <tr>
      <th style="text-align: center"><strong>Experiment</strong></th>
      <th style="text-align: center">1</th>
      <th style="text-align: center">2</th>
      <th style="text-align: center">3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center"><strong>Sentence</strong></td>
      <td style="text-align: center">Example 1</td>
      <td style="text-align: center">Example 2</td>
      <td style="text-align: center">Example 3</td>
    </tr>
    <tr>
      <td style="text-align: center"><strong>Errors</strong></td>
      <td style="text-align: center">error A, error B, error C</td>
      <td style="text-align: center">error C</td>
      <td style="text-align: center">error B</td>
    </tr>
  </tbody>
  <caption>Table 1. This is Table 1's caption</caption>
</table>
<br>
<div style="text-align: center;">
<img style="height: 300px;" alt="" src="./files/results.png">
</div>
<br><br>

<hr>



<h2 id="conclusion">Conclustion and Future Work</h2>
<p>

  How easily are your results able to be reproduced by others?
  Did your dataset or annotation affect other people's choice of research or development projects to undertake?
  Does your work have potential harm or risk to our society? What kinds? If so, how can you address them?
  What limitations does your model have? How can you extend your work for future research?</p>


<hr>


  </div>
  


</body></html>
```