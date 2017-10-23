---
title: "Transfer Defect Learning Using Dependency Clustering"
bibliography: config/library.bib
abstract:  Preventing the introduction of software defects at commit-time is a growing line of research in the software maintenance community. Existing approaches leverage code and process metrics to build statistical models that can effectively prevent defect insertion and propose fixes in a software project. Metrics, however, may vary from one project to another, hindering the reuse of these models. Moreover, these techniques operate within single projects despite the fact that many projects share dependencies and are, therefore, vulnerable to similar faults. In this paper, we propose a novel approach, called BIANCA, which relies on clone detection and dependency analysis to detect _risky_ commits within and across related projects. When applied to 42 projects, BIANCA achieves an average precision, recall, and F-measure of 90.75%, 37.15%, and 52.72%, respectively. We also found that only 8.6% of the risky commits detected by BIANCA match other commits from the same project, suggesting that dependencies among projects need to be considered for effective prevention of risky commits. In addition, BIANCA is able to propose qualitative fixes to transform _risky_ commits into _non-risky_ ones in 78.67% of the cases. 

author: 
- name: Mathieu Nayrolles,  Abdelwahab Hamou-Lhadj
  affiliation: SBA Lab, ECE Dept, Concordia University
  location: Montréal, QC, Canada
  email: \{mathieu.nayrolles, wahab.hamou-lhadj\}\@concordia.ca
csl: config/ieee.csl
classoption: 
- 10pt
- conference
keyword: 
- Bug Prediction
- Risky Software Commits
- Clone Detection
- Software Maintenance
---

\IEEEraisesectionheading{\section{Introduction}\label{sec:introduction}}

\IEEEPARstart{R}{esearch} in software maintenance has evolved over the years to include areas like mining bug repositories, bug analytic, and bug prevention and reproduction. The ultimate goal is to develop better techniques and tools to help software developers detect, correct, and prevent bugs in an effective and efficient manner. 

One particular (and growing) line of research focuses on the problem of preventing the introduction of bugs by detecting risky commits (preferably before the commits reach the central repository). Recent approaches (e.g., [@Lo2013; @Nam2013]) rely on training models based on code and process metrics (e.g., code complexity, experience of the developers, etc.) that are used to classify new commits as risky or not. Metrics, however, may vary from one project to another, hindering the reuse of these models. Consequently, these techniques tend to operate within single projects only, despite the fact that many large projects share dependencies, such as the reuse of common libraries. This makes them potentially vulnerable to similar faults. 


Another advantage of BIANCA is that it uses commits that are used to fix previous defect-introducing commits to guide the developers on how to improve risky commits. This way, BIANCA goes one step further than existing techniques by providing developers with a potential fix for their risky commits.

We validated the performance of BIANCA on 42 open source projects, obtained from Github. The examined projects vary in size, domain and popularity.


# The BIANCA Approach {#sec:bianca}



\input{tex/approach}


## Clustering Project Repositories {#sec:clustering}

We cluster projects according to their dependencies.  The rationale is that projects that share dependencies are most likely to contain defects caused by misuse of these dependencies. In this step, the project dependencies are analysed and saved into a single NoSQL graph database as shown in Figure \ref{fig:bianca3}. Graph databases use graph structures as a way to store and query information.  In our case,  a node corresponds to a project that is connected to other projects on which it depends. Project dependencies can be automatically retrieved if projects use a dependency manager such as Maven. 

Figure \ref{fig:network-sample} shows a simplified view of a dependency graph for a project named \texttt{com.badlogicgames.gdx}.
As we can see, \texttt{com.badlogicgames.gdx} depends on projects owned by the same organization (i.e., badlogicgames) and other organizations such as Google, Apple, and Github.

\input{tex/network-sample}

Once the project dependency graph is extracted, we use a clustering algorithm to partition the graph. To this end, we choose the Girvan–Newman algorithm [@Girvan2002; @Newman2004], used to detect communities by progressively removing edges from the original network. Instead of trying to construct a measure that identifies the edges that are the most central to communities, the Girvan–Newman algorithm focuses on edges that are most likely "between" communities. This algorithm is very effective at discovering community structure in both computer-generated and real-world network data [@Newman2004]. Other clustering algorithms can also be used. 

## Building a Database of Code Blocks of Defect-Commits and Fix-Commits {#sec:offline}

To build our database of code blocks that are related to defect-commits and fix-commits, we first need to identify the respective commits. Then, we extract the relevant blocks of code from the commits.

**Extracting Commits:** BIANCA listens to bug (or issue) closing events happening on the project tracking system. Every time an issue is closed, BIANCA retrieves the commit that was used to fix the issue (the fix-commit) as well as the one that introduced the defect (the defect-commit). Retrieving fix-commits, however, is known to be a challenging task [@Wu2011]. This is because the link between the project tracking system and the code version control system is not always explicit. In an ideal situation, developers would add a reference to the issue they work on inside the description of the commit. However, this good practice is not always followed. To make the link between fix-commits and their related issues, we turn to a modified version of the back-end of commit-guru [@Rosen2015]. Commit-guru is a tool, developed by Rosen _et al._  [@Rosen2015] to detect _risky commits_. In order to identify risky commits, Commit-guru builds a statistical model using change metrics (i.e.,  amount of lines added, amount of lines deleted, amount of files modified, etc.) from past commits known to have introduced defects in the past.

Commit-guru's back-end has three major components: ingestion, analysis, and prediction. We reuse the ingestion and the analysis part for BIANCA. The ingestion component is responsible for ingesting (i.e., downloading) a given repository.
Once the repository is entirely downloaded on a local server, each commit history is analysed. Commits are classified using the list of keywords proposed by Hindle *et al.* [@Hindle2008]. Commit-guru implements the SZZ algorithm [@Kim2006c] to detect risky changes, where it performs the SCM blame/annotate function on all the modified lines of code for their corresponding files on the fix-commit's parents. This returns the commits that previously modified these lines of code and are flagged as the defect introducing commits (i.e., the defect-commits). Prior work showed that commit-guru is effective in identifying defect-commits and their corresponding fixing commits [@Kamei2013a] and the SZZ algorithm, used by commit-guru, is shown to be effective in detecting risky commits [@Kamei2013; @Rosen2015]. Note that we could use a simpler and more established approach such as Relink [@Wu2011] to link the commits to their issues and re-implement the classification proposed by Hindle *et al.* [@Hindle2008] on top of it. However, commit-guru has the advantage of being open-source, making it possible to modify it to fit our needs by fine-tuning its performance.


# Case Study Setup {#sec:exp}

In this section, we present the setup of our case study in terms of repository selection, dependency analysis, comparison process and evaluation measures.

## Project Repository Selection {#sec:rep}

To select the projects used to evaluate our approach, we followed three simple criteria. First, the projects need to be in Java and use Maven to manage dependencies. This way, we can automatically extract the dependencies and perform the clustering of projects. The second criterion is to have projects that enjoy a large community support and interest. We selected projects that have at least 2000 followers. Finally, the projects must have a public issue repository to be able to mine their past issues and the fixes. We queried Github with these criteria and retrieved 42 projects (see Table \ref{tab:results} for the list of projects), including those from some of major open-source contributors such as Alibaba, Apache Software Foundation, Eclipse, Facebook, Google and Square. 

## Project Dependency Analysis {#sec:dependencies}

Figure \ref{fig:dep-graph} shows the project dependency graph. The dependency graph is composed of 592 nodes divided into five clusters shown in yellow, red, green, purple and blue. The size of the nodes in Figure \ref{fig:dep-graph} is proportional to the number of connections from and to the other nodes.

\input{tex/dependencies}

As shown in Figure \ref{fig:dep-graph},  these Github projects are very much interconnected. On average, the projects composing our dataset have 77 dependencies. Among the 77 dependencies, on average, 62 dependencies are shared with at least one other project from our dataset.

Table \ref{tab:communities} shows the result of the Girvan–Newman clustering algorithm  in terms of centroids and betweenness. The blue cluster is dominated by Storm from The Apache Software Foundation. Storm is a distributed real-time computation system. Druid by Alibaba, the e-commerce company that provides consumer-to-consumer, business-to-consumer and business-to-business sales services via web portals, dominates the yellow cluster. In recent years, Alibaba has become an active member of the open-source community by making some of its projects publicly available. The red cluster has Hadoop by the Apache Software Foundation as its centroid. Hadoop is an open-source software framework for distributed storage and distributed processing of very large datasets on computer clusters built from commodity hardware. The green cluster is dominated by the Persistence project of OpenHab.  OpenHab proposes home automation solutions and the Persistence project is their data access layer. Finally, the purple cluster is dominated by Libdx by Badlogicgames, which is a cross-platform framework for game development.

A review of each cluster shows that this partitioning divides projects in terms of high-level functionalities. For example, the blue cluster is almost entirely composed of projects from the Apache Software Foundation. Projects from the Apache Software Foundation tend to build on top of one another. We also have the red cluster for Hadoop, which is by itself an ecosystem inside the Apache Software Foundation. Finally, we obtained a cluster for e-commerce applications (yellow), real-time network application for home automation (green), and game development (purple).


\input{tex/clusters}


## Building a Database of Defect-Commits and Fix-Commits for Performances Evaluation {#sub:golden}

To build the database that we can use to assess the performance of BIANCA, we use the same process as discussed in Section \ref{sec:offline}. We used Commit-guru to retrieve the complete history of each project and label commits as defect-commits if they appear to be linked to a closed issue. The process used by Commit-guru to identify commits that introduce a defect is simple and reliable in terms of accuracy and computation time [@Kamei2013]. We use the commit-guru labels as the baseline to compute the precision and recall of BIANCA. Each time BIANCA classifies a commit as _risky_, we can check if the _risky_ commit is in the database of defect-introducing commits. The same evaluation process is used by related studies  [@ElEmam2001; @Lee2011a; @Bhattacharya2011; @Kpodjedo2010].

## Process of Comparing New Commits {#sec:newcommits}

Because our approach relies on commit pre-hooks to detect risky commits, we had to find a way to *replay* past commits. 
To do so, we *cloned* our test subjects, and then created a new branch called *BIANCA*. When created, this branch is reinitialized at the initial state of the project (the first commit) and each commit can be replayed as they have originally been.  For each commit, we store the time taken for *BIANCA* to run, the number of detected clone pairs, and the commits that match the current commit. 
As an example, let's assume that we have three commits from two projects. 
At time $t_1$,  commit $c_1$ in project $p_1$ introduces a defect. 
The defect is experienced by an user that reports it via an issue $i_1$ at $t_2$.
A developer fixes the defect introduced by $c_1$ in commit $c_2$ and closes $i_1$ at $t_3$.
From $t_3$ we known that $c_1$ introduced a defect using the process described in Section \ref{sub:golden}.
If at $t_4$, $c_3$ is pushed to $p_2$ and $c_3$ matches $c_1$ after preprocessing, pretty-printing and formatting, then $c_3$ is classified as _risky_ by BIANCA and $c_2$ is proposed to the developer as a potential solution for the defect introduced in $c_3$.


## Evaluation Measures

Similar to prior work focusing on risky commits (e.g., [@SunghunKim2008; @Kamei2013]), we used precision, recall, and F-measure to evaluate our approach. They are computed using TP (true positives), FP (false positives), FN (false negatives), which are defined as follows:


- TP:  is the number of defect-commits that were properly classified by BIANCA
- FP:  is the number of healthy commits that were classified by BIANCA as risky
- FN:  is the number of defect introducing-commits that were not detected by BIANCA
- Precision: TP / (TP + FP)
- Recall: TP / (TP + FN)
- F-measure: 2.(precision.recall)/(precision+recall)

It is worth mentioning that, in the case of defect prevention, false positives can be hard to identify as the defects could be in the code but not yet reported through a bug report (or issue). To address this, we did not include the last six months of history. Following similar studies [@Rosen2015; @Chen2014; @Shivaji2013; @Kamei2013b], if a defect is not reported within six months then it is not considered.

# Case Study Results {#sec:result}






## Baseline Classifier Comparison

Although our average F-measure of 52.72% may seem low at first glance, achieving a high F-measure for unbalanced data is very difficult [@menzies2007problems]. Therefore, a common approach to ground detection results is to compare it to a simple baseline.

To the best of our knowledge, this is the first approach that relies on  code similarity instead of code or process metrics for the detection of risky commits. Comparing it to other approaches will not be accurate. In addition, existing metric-based techniques (e.g., [@Nam2013]) detect risky commits within single projects only. BIANCA, on the other hand, operates across projects. We compared BIANCA  with a random classifier to have a baseline and show that we perform better than a simple baseline.

The baseline classifier first generates a random number $n$ between 0 and 1 for the 165,912 commits composing our dataset.
For each commit, if $n$ is greater than 0.5, then the commit is classified as risky and vice versa. As expected by a random classifier, our implementation detected ~50% (82,384 commits) of the commits to be _risky_. It is worth mentioning that the random classifier achieved 24.9% precision, 49.96% recall and 33.24% F-measure. Since our data is unbalanced (i.e., there are many more _healthy_ than _risky_ commits) these numbers are to be expected for a random classifier. Indeed, the recall is very close to 50% since a commit can take on one of two classifications, risky or non-risky. While analysing the precision, however, we can see that the data is unbalanced (a random classifier would achieve a precision of 50% on a balanced dataset).

It is important to note that the purpose of this analysis is not to say that we outperform a simple random classifier, rather to shed light on the fact that our dataset is unbalanced and achieving an average F-= 52.72% is non-trivial, especially when a baseline only achieves an F-measure of 33.24%.


# Discussion {#sec:threats}

In this section we propose a discussion on limitations and threats to validity.

## Limitations

## Threats to Validity


# Related Work {#sec:relwork}
The work most related to ours come from two main areas, work that aims to predict future defects in files, modules and changes and work that aims to propose or generate patches for buggy software.

## File, Module and Risky Change Prediction

The majority of previous file/module-level prediction work used code or process metrics.
Approaches using code metrics only use information from the code itself and do not use any historical data. Chidamber and Kemerer published the well-known CK metrics suite [@Chidamber1994] for object oriented designs and inspired Moha *et al.* to publish similar metrics for service-oriented programs [@Moha]. Another famous metric suite for assessing the quality of a given software design is Briand's coupling metrics [@Briand1999a].

The CK and Briand’s metrics suites have been used, for example, by Basili *et al.* [@Basili1996], El Emam *et al.* [@ElEmam2001], Subramanyam *et al.* [@Subramanyam2003] and Gyimothy *et al.* [@Gyimothy2005] for object-oriented designs. 
Service oriented designs have been far less studied than object oriented design as they are relatively new, but, Nayrolles *et al.* [@Nayrolles; @Nayrolles2013a], Demange *et al.* [@demange2013] and Palma *et al.* [@Palma2013] used Moha *et al.* metric suites to detect software defects.
All these approaches, proved software metrics to be useful at detecting software fault for object oriented and service oriented designs, respectively. More recently, Nagappan *et al.* [@Nagappan2005; @Nagappan2006] and Zimmerman *et al.* [@Zimmermann2007; @Zimmermann2008] further refined metrics-based detection by using statical analysis and call-graph analysis.

Other approaches use historical development data, often referred to as process metrics. Naggapan and Ball [@Nagappan] studied the feasibility of using relative churn metrics to prediction buggy modules in the Windows Server 2003. Other work by Hassan *et al* and Ostrand *et al* used past changes and defects to predict buggy locations (e.g., [@Hassan2005], [@Ostrand2005]). Hassan and Holt proposed an approach that highlights the top ten most susceptible locations to have a bug using heuristics based on file-level metrics [@Hassan2005]. They find that locations that have been recently modified and fixed locations are the most defect-prone. Similarly, Ostrand *et al.* [@Ostrand2005] predict future crash location by combining the data from changed and past defect locations. They validate their approach on industrial systems at AT&T. They showed that data from prior changes and defects can effectively defect-prone locations for open-source and industrial systems. Kim *et al.* [@Kim2007a] proposed the bug cache approach, which is an improved technique over Hassan and Holt's approach [@Hassan2005]. Rahman and Devanbu found that, in general, process-based metrics perform as good as or better than code-based metrics [@rahman2013].

Other work focused on the prediction of risky changes. Kim et al. proposed the change classification problem, which predicts whether a change is buggy or clean [@SunghunKim2008]. Hassan [@Hassan2009] used the entropy of changes to predict risky changes. They find that the more complex a change is, the more likely it is to introduce a defect. Kamei *et al.* performed a large-scale empirical study on change classification [@Kamei2013]. They aforementioned studies find that size of a change and the history of the files being changed (i.e., how buggy they were in the past) are the best indicators of risky changes.

Our work shares a similar goal to the work on the prediction of risky changes, however, BIANCA takes a different approach in that it leverages dependencies of a project to determine risky changes.

Our work differs from the work on automated patch generation in that we do not generate patches, rather we use clone detection to determine the similarity of a change to a previous risky change and suggest to the developer the fixes of the prior risky changes.


# Conclusion {#sec:conclusion}

In this paper, we presented BIANCA (Bug Insertion ANticipation by Clone Analysis at commit time), an approach that detects risky commits (i.e., a commit that is likely to introduce a bug) with  an average of 90.75% precision and 37.15% recall.
BIANCA uses clone detection techniques and project dependency analysis to detect risky commits within and across projects.  BIANCA operates at commit-time, i.e., before the commits  reach the code central repository. In addition, because it relies on code comparison, BIANCA does not only detect risky commits but also makes recommendations to developers on how to fix them. We believe that this makes BIANCA a practical approach for preventing bugs and proposing corrective measures that integrates well with the developers workflow through the commit mechanism.  

To build on this work, we need to conduct a human study with developers in order to gather their feedback on the approach. 
The feedback obtained will help us fine-tune the approach. Also, we want to examine the relationship between project cluster measures (such as betweenness) and the performance of BIANCA. Finally, another improvement to BIANCA would be to support Type 4 clones.

# Reproduction Package & Dataset

As described in Section \ref{sec:newcommits}, we rely heavily on virtual machines instrumentation and coordination to run our experiments. Providing a straightforward reproduction package is therefore very challenging.
However, we are happy to share our consolidated dataset: https://github.com/MathieuNls/tdl-data.
The dataset is composed of three compressed PostgresSQL formatted tables: clones, commits and repository.
The clone table stores the relationship between set of similar commits.
The commits themselves are in the commit table with details about their author, repository, commit message and all the metrics found in commit guru [@Rosen2015].
Finally, the repository table describes the repository used in terms of url, name and ingestion status.


\newpage

\input{tex/result}	

\newpage	

\section*{References} 
\small
\setlength{\parindent}{0pt}
\setlength{\parskip}{6pt plus 2pt minus 1pt}
