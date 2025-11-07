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

Intro presentation on our project:
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

Full project proposal report:
```latex
% This must be in the first 5 lines to tell arXiv to use pdfLaTeX, which is strongly recommended. The hyperref package requires pdfLaTeX in order to break URLs across lines.
\pdfoutput=1
\documentclass[11pt]{article}


\usepackage[final]{acl} % \usepackage[final\review\preprint]{acl}
\usepackage{times}
\usepackage{latexsym}
\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{microtype}
\usepackage{inconsolata}
\usepackage{graphicx}
\usepackage{listings} % for code formatting
\renewcommand{\lstlistingname}{Code} 
\lstset{basicstyle=\ttfamily\small, breaklines=true, breakatwhitespace=true, columns=fullflexible, keepspaces=true, showstringspaces=false, frame=tb, tabsize=2, numbers=left, numberstyle=\tiny, language=Python}

%\setlength\titlebox{<dim>}
\usepackage{fancyhdr}


\title{Appropriateness Filtering for Corporate RAG Database Storage}

\author{
  Team NetWatch: A.Berg, Z.Johnson, A.Slinger, and S.Subramanian \\
  \texttt{ber00221@umn.edu, joh15514@umn.edu, sling031@umn.edu, subra287@umn.edu}
}

\begin{document}

\pagestyle{fancy}
\fancyhead{} % clear all header fields
\fancyhead[R]{\small{CSCI 5541 NLP S25}}
\fancyhead[L]{\small{Project Proposal}}
\fancyfoot{}
\fancyfoot[L]{NetWatch} % clear all footer fields
\fancyfoot[R]{\small\thepage}

\maketitle
\begin{abstract}
Retrieval-Augmented Generation (`RAG') systems rely on the quality of content stored in their vector database, but without care data ingestion can easily sweep up inappropriate information (including personal, toxic, or irrelevant data). Given some of the richest sources of a business's internal information are informal channels (such as email or Slack) where wanted and non-wanted content often co-occur within a single message, disambiguating between information is an important but non-trivial challenge. This project introduces a novel pre-embedding filtering framework to evaluate and classify `informal business' communication claims before they enter the RAG store, leveraging contextual features available at ingestion time. We also propose an evaluation strategy that aids wider understanding of both 1) the compounding effects of a multilayer filter approach and 2) using custom classifier modules vs. LLM prompting for our specific (non-public) domain. 

\end{abstract}

\section{Introduction}
Modern organizations increasingly rely on Retrieval-Augmented Generation (`RAG') systems to access institutional knowledge from large internal corpora. Yet over 90\% of enterprise information remains unstructured, such as in emails, chat logs, and meeting transcriptions \citep{mckinsey2024data} - with the average knowledge worker receiving 117 emails per day, many skimmed in under a minute \citep{microsoft2025infinite}. Corporate communications therefore represent a rich, but risky, source of unstructured information. Given the mixed personal and business nature of these sources, companies risk embedding sensitive, irrelevant, or unreliable information into their knowledge bases, alongside the valuable information they intend to capture. This `inappropriate' information can degrade RAG performance \citep{amiraz2025distracting} and create compliance concerns.

Existing RAG research tends to concentrate on post-retrieval techniques such as graph-based retrieval or reciprocal-rank fusion to improve question-answering (`QA') quality \citep{yang2024crag}. However, relatively little attention has been given to the quality of content that populates the vector database itself. Our project investigates the effectiveness of pre-embedding filtering, considering QA quality when limiting what enters the RAG store in the first place. We hypothesize that filtering out inappropriate messages at their most context-rich stage (raw email text, headers, and thread metadata) can simultaneously improve retrieval relevance and prevent private or low-value content from being stored.

We propose a modular pipeline that decomposes corporate emails into claim-level statements and applies a suite of fine-tuned classifier modules to assess each claim’s appropriateness, reliability, and informational value. Modules will detect issues such as Personally Identifiable Information (`PII') leakage, compliance risk, and low-information content like sarcasm or hyperbole \citep{hassan2017toward}. Ambiguous statements that blend sarcasm with genuine updates may be retained but annotated with confidence scores, allowing downstream retrieval to weight rather than discard them. Filtered claims will be then be embedded and stored for later retrieval by our RAG agent.  

Our goal is to test whether pre-storage filtering improves both retrieval quality and ethical safety compared to standard RAG baselines. Using the ENRON email corpus as our primary dataset, we will construct augmented evaluation sets that combine professional and non-professional content to measure improvements in answer relevance and data integrity. This proposal defines an early research framework for appropriateness-aware ingestion as a novel stage in corporate RAG pipelines.

\section{Literature Survey}

Retrieval-Augmented Generation (RAG) systems have emerged as a dominant framework for combining large language models with external knowledge sources. The seminal work by \citet{lewis2020rag} introduced RAG as a means of improving factual consistency in open-domain QA by integrating dense retrieval with generative reasoning. Subsequent efforts, such as Meta's Comprehensive RAG Benchmark (`CRAG') proposed by \citet{yang2024crag}, standardized RAG evaluation across diverse document domains. Contemporary analyses of RAG design choices, such as \citet{wang2024ragbest}, emphasize that most performance gains have focused on post-retrieval optimization through methods such as ranking and graph-based fusion, while comparatively little attention has been paid to the pre-embedding stage of data ingestion. Our work directly explores this gap by examining whether filtering unstructured corporate communications prior to embedding can improve both retrieval relevance and ethical data compliance.

A core element of our methodology is claim-level decomposition, in which each email is segmented into smaller factual statements. \citet{hassan2017claimbuster} formalized claim detection as a binary classification task for identifying check-worthy statements, while follow-up work such as Microsoft’s Claimify approach \citep{metropolitansky2025towards} extended this to structured claim extraction. These studies collectively motivate our use of large language models to transform informal email discourse into claim-level representations suitable for subsequent semantic filtering.

Our experimental domain builds upon established corporate communication datasets. The Enron Email Corpus introduced by \citet{klimtyang2004enron} remains a foundational resource for studying organizational communication, linguistic tone, and metadata-rich correspondence. Further, the Avocado Research Email Collection \citep{oard2015avocado} also provides a common complementary dataset with extended thread-level annotations (though with an often prohibitive license cost). However both of these datasets are labeled at the email (or inter-email) level, a restriction we discuss further below.

Filtering of claims for appropriateness, reliability, and informational value draws on advances in sentiment, sarcasm, and compliance detection. Transformer-based models for sarcasm recognition have demonstrated robust performance on conversational text \citep{khodak2018sarcasm, joshi2022sarcasm}, while industry literature underscores the necessity of filtering personally identifiable or sensitive information at ingestion time \citep{aws2024filter}. Together, these works inform our classifier modules that detect low-information or non-compliant claims before embedding. We also contrast this `trained model' approach with direct use of generalist LLMs for scoring (both zero-shot and few-shot). For our the few-shot tests we intend to follow recent work on reducing prompt-quality sensitivity \citep{khattab2023dspy} to provide fairer benchmarks.

\section{Methodology}

This project will develop and evaluate a modular pipeline for pre-filtering corporate communications before they are stored in a database for a RAG system. Our four-stage pipeline (Figure \ref{fig:RAG_stages}) includes data ingestion, claim filtering, vector storage/retrieval, and system evaluation, implemented within a Colab-based RAG framework.

\begin{figure}[!h]
    \centering
    \includegraphics[width=0.75\linewidth]{RAG_stages.png}
    \caption{Project stages}
    \label{fig:RAG_stages}
\end{figure}

The initial step involves ingesting unstructured, informal text from corporate communication sources. During the demo, we plan to have a Gmail API set up for real-time email ingestion, but for the majority of our testing, we will ingest data from the ENRON email database. During ingestion, each message will be broken down into individual claims, filtered and then (non-discarded) claims will be embedded and stored in a Pinecone vector database\footnote{See: \url{https://www.pinecone.io/product/}}.

Each claim will be scored using layered fine-tuned classifier modules (e.g., sarcasm, hyperbole, or formality - as discussed in Section 3.1), where below threshold claims are immediately discarded, while non-discarded claims will retain the classifiers' metadata scores for later contextual retrieval.

Retrieval will be implemented with a simple approximate-nearest-neighbors vector search of the embedded claims to find relevant claims. We will also test a Reciprocal-Rank-Fusion (RRF) of the neighbor space, according to the semantic labels generated in the last phase. RRF should help to further prioritize relevant claims, allowing some contextual aid in the absence of a perfect supporting claim. The supporting claims data will then be fed into the final LLM, in charge of summarizing the information from the claims and presenting this to the user. All of this will be done using LangChain, implementing vector search and ranking as a LangChain Tool, and the final LLM as a LangChain agent.

The final stage of implementation is evaluation, comparing the relevance of the results generated by semantic ranking to results from a one-shot LLM approach, multi-filter LLM approach, and more, as discussed further in Section 4.

\subsection{Proposed filter modules}
As discussed, informal business communications are littered with non-relevant and/or inappropriate information for a corporate knowledge database. To counteract this, we plan to implement between 4-7 filter modules, prior to vector database embedding, where rankings will be used to both cull claims immediately and to rank fetched information when retrieving data. 

The potential filters that we will consider implementing are as follows:
\begin{itemize}
    \item \textbf{Confidentiality:} To detect confidential information, such as HR data (salaries, medical), or `Material Nonpublic Information' (market moving info) that needs specific handling.
    \item \textbf{Personal info:} To detect both regulated personal info (Sensitive PII - such as religion, union membership, etc) and non regulated but non-relevant info (such as events, holiday plans, relationships, etc).
    \item \textbf{Sarcasm:} To detect sarcasm/satire markers that indicate a claim/statement is not meant to be taken literally, and so should not be used to ground a RAG's generated responses.
    \item \textbf{Speculation/Opinion:} To flag language that is explicitly non-factual, such as predictions, opinions, or hypotheticals ("I think sales might double next year") - to prevent RAG generated responses presenting subjective beliefs as established facts.
    \item \textbf{Toxicity:} To identify profanity, harassment, and other unprofessional language - ensuring generated responses are not based on or repeat inappropriate communications.
    \item \textbf{Inconsistency:} To identify statements that may negate existing database facts, or provide a stale/expired view of items already in the database.  
    \item \textbf{Relevance:} To detect auto generated content, such as circulars, system logs or `config dumps' - and other types of non-relevant messages for corporate knowledge.
\end{itemize}

\subsection{Novelty consideration}
Based on our proposed methods and testing we see three key novel aspects of this project we discuss in turn.

\textbf{Pre-embedding filtering:} Unlike prior RAG research focused on retrieval optimization (e.g., GraphRAG, RRF), we evaluate pre-embedding filtering at ingestion, where full context is available - aiming to ensure inappropriate information never enters a company's knowledge database.  While this aligns with established best practice for security reasons \citep{aws2024filter}, limited work has considered the direct benefit of this approach for QA quality vs. post-embedding filtering.

\textbf{Informal business communications domain:} Most RAG research, such as the CRAG Benchmark \citep{yang2024crag}, tends to consider public sources, pre-structured knowledge databases, or formal business documentation (e.g. PDFs, spreadsheets, etc). Our work specifically focuses on a domain where both appropriate and non-appropriate content often co-exist within a single message. Primary research data sources for this domain (such as ENRON and Avocado email datasets) are labeled at the message level, so to conduct intra-message filtering we propose a new synthetic composite dataset discussed in Section 4.

\textbf{End-to-end ablation study:} Finally, our evaluation will compare multiple layered Language Models vs prompted LLMs for business appropriateness and QA quality. This includes an ablation study of filter layers to understand compounding effects. While RAG ablation studies exist (for example \citet{lewis2020rag}), to our knowledge there is limited prior work that conducts such evaluation in the pre-embedding vs. post-retrieval filtering on intra-message email content.



\section{Testing and Evaluation} 
Our project seeks to demonstrate a new approach to solve the business challenge of extracting valuable and appropriate info from business emails, Slack, etc., while discarding unwanted information. To evaluate the performance of our proposed `pre-storage filtering system' on RAG-QA, we intend to utilize a multi-stage testing framework (primarily based on a modified ENRON email corpus) to measure the impact/effectiveness of each filtering module on downstream RAG performance.

To achieve this our testing will have two main approaches:
\begin{enumerate}
    \item \textbf{Overall performance:} An ablation-style analysis, where we systematically remove modules (classifiers and claims) to quantify their contribution to overall system quality - until we reach our prior Vanilla (no filters) RAG status.
    \item \textbf{Module level performance:} A benchmarking of each classifier filter's performance, compares to the accuracy using LLM prompting (both zero-shot and few-shot). If we have time we also intend to ensure we are comparing to optimal prompts, by using DSPy framework's prompt optimization approach.
\end{enumerate}

Both of these approaches will use evaluation and training datasets based primarily on the categorical labeled ENRON email dataset \citep{UCB2004EnronAnnotated}, which has c.1,700 labeled emails with tone and topic annotations. This dataset has labels at the email level, so to train/test intra-email classification, we also intend to create an new synthetic composite dataset, based on seeding `professional facts' into ENRON emails that have been labeled as having `non-professional' content. Our professional facts will be taken from the Microsoft `MeetingBank-QA-Summary' dataset \citep{microsoft2024MeetingBankQA}, which has c.860 meeting transcripts with 3 curated QA pairs each \citep{pan2024llmlingua}. Inserting these transcripts between existing paragraphs, guarantees a mix of context, while also providing us with a QA pair sets that we can use to check the output of our RAG.

To test our individual classifiers, we will follow a similar approach of adapting the ENRON dataset, but instead poison emails classified as fully professional (i.e. by both Berkeley student labelers) with a paragraph of non-professional information - and then check if any claims containing this information survive our filtering.

\subsection{Overall performance}
We intend to perform an ablation analysis to isolate the effect of each classifier on the overall quality of the knowledge base - specifically considering a Vanilla RAG baseline system, vs. incremental filter stacking (i.e. the sequential addition of filters in a logical order of corporate relevance. Each configuration will be evaluated using the same retrieval and query test set to ensure comparability. These ablation tests should demonstrate if additional filters improve our system's quality or risks over-filtering wanted information.

\subsection{Module performance}
To clarify the success of the claim extraction module (1b in Figure \ref{fig:RAG_stages}), given this is already LLM driven, we will not compare it to an LLM prompt but rather intend to benchmark on the Microsoft Claimify dataset \citep{microsoft2024claimify} to ensure correct accuracy and operation.

To assess the performance of our specialized classifier modules vs. direct LLM-based filtering, we will run tests of how the classifiers perform (based on accuracy of filtering) vs. a GPT-5/GPT-4o zero-shot and few-shot prompts.

\section{Project Supporting Items}

In addition to the above project proposal, we have also provided in Appendix~\ref{sec:appendix_A} specific details on how we have addressed feedback from the pitch presentation, and in Appendix~\ref{sec:appendix_B} we provide an expected role assignment for each project member.


\newpage
\onecolumn  
\bibliography{custom}


\newpage
\appendix
\section{Addressing feedback from the pitch presentation}
\label{sec:appendix_A}

The below list addresses the key items raised as feedback during the proposal presentation, and how our updated proposal addresses each:

\begin{itemize}
    \item \textbf{Professor's comment:} Please include a comparison for each layer vs. a direct zero-shot/few-shot LLM prompt approach - and consider the effectiveness of this being layered vs. the multilayer classifier architecture.
    \begin{itemize}
      \item \textbf{Response:} We have adjusted our project evaluation to capture this. We understood from the comment that if the project overruns the `layered direct LLM approach, plus ablation study' would be considered more interesting than the classifier training - so we will weight our project plan to achieving this element first (though as described above hope to complete both parts).
    \end{itemize}
            
    \item \textbf{TA comment:} ``The main critique was a lack of clear motivation and goal definition".
    \begin{itemize}
      \item \textbf{Response:} The commercial motivation and goal for the project have been detailed in the introduction, and the academic motivation (i.e. novelty) is detailed in Section 3.2. We consider why companies need this type of tool (significant valuable information existing in email but needing structuring), why the information needs filtering (RAG distraction issues) prior to embedding (privacy and best context point), and why this in non-trivial in this domain (both appropriate and inappropriate information co-existing within one message). 
    \end{itemize}
    
    \item \textbf{TA question:} ``why sentiment, sarcasm, or emotional tone are relevant in professional communication contexts" and ``more concrete categories, use cases, and justification for the filtering approach"?
    \begin{itemize}
      \item \textbf{Response:} As detailed above, given informal business communication typically contains co-occurring personal and professional items, our filters (such as sentiment or emotional tone) are designing to to detect markers that the claim being reviewed may be non-professional in nature and so should be discarded prior to embedding. Specific filters we intend to implement, and the reason for each, is detailed in Section 3.1.
    \end{itemize}

    \item \textbf{TA question:} ``how ambiguous cases (e.g., sarcasm embedded in important content) would be handled without discarding useful information"?
    \begin{itemize}
      \item \textbf{Response:} As detailed above, breaking a message into claims (i.e. sub-topics of the message) should help us separate parts of the email that may have different context - and save or discard these parts independently. Where an item truly can't be disambiguated, we are currently proposing it would be stored - but with filter context saved as metadata to help appropriate usage/ranking at retrieval time.
    \end{itemize}
        
    \item \textbf{TA question:} ``Specific labeling strategies and evaluation methods were requested" and ``comparisons showing improved retrieval quality over vanilla RAG"
    \begin{itemize}
      \item \textbf{Response:} In the above proposal we detail both our multilayer labeling strategy, including a hard rejection threshold and a metadata augmentation for retrial time ranking, and our proposed evaluation methods, i.e. comparison with LLM and ablation study. Specifically we have designed our experiment to evaluate the efficacy of retrieval from pre-filtered data vs. vanilla RAG as described.
    \end{itemize}
    
\end{itemize}

\newpage
\section{Role assignment}
\label{sec:appendix_B}

The below table addresses the proposed role assignment for the members of Team Netwatch - though these are expected to evolve based on challenges encountered during the project.

\begin{table}[!h]
    \centering
    \begin{tabular}{lp{0.7\textwidth}}
    \hline
    \textbf{Member} & \textbf{Area of focus during the project}\\
    \hline
    A.Berg &            Vanilla + RRF RAG. LangChain flow. Gmail API. \\
    Z.Johnson &         Module development. Module vs. LLM testing. \\
    A.Slinger &         New dataset development. DSPy prompt optimization\\
    S.Subramanian &     Module development. Ablation testing. \\
    \hline
    \end{tabular}
    \caption{Role assignment for NetWatch team members}
    \label{tab:role_assignment}
\end{table}


\end{document}


\subsection{Citations}

\citet{xxx} \\
\citep{xxx} \\
\citealp{xxx} \\
\citeyearpar{xxx} \\
\citeposs{xxx} \\
\footnote{See: \url{ https://xxx}}
Table~\ref{tab:name}
Code~\ref{lst:xxx}

\begin{lstlisting}[language=Python, caption={xxx}, label={lst:xxx}][!h]
    def code_example():
        pass
\end{lstlisting}


\begin{figure}[!h]
    \centering
    \includegraphics[width=1\linewidth]{XXX}

    \centering
    \includegraphics[width=1\linewidth]{XXX}

    \caption{XXX}
    \label{fig:XXX}
\end{figure}

% \newpage
% \onecolumn  
% \section{Appendix: XXX}
% \label{sec:XXXX}
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