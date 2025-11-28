
![Text Box: arXiv:2407.02818v1 [cs.SE] 3 Jul 2024](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image001.gif)**WizardMerge - Save Us From Merging Without Any Clues**

QINGYU ZHANG, The University of Hong Kong, China

JUNZHE LI, The University of Hong Kong, China JIAYI LIN, The University of Hong Kong, China JIE DING, The University of Hong Kong, China

LANTENG LIN, Beijing University of Posts and Telecommunications, China

CHENXIONG QIAN∗, The University of Hong Kong, China

Modern software development necessitates efficient version-oriented collaboration among developers. While Git is the most popular version control system, it generates unsatisfactory version merging results due to textual-based workflow, leading to potentially unexpected results in the merged version of the project. Although numerous merging tools have been proposed for improving merge results, developers remain struggling to resolve the conflicts and fix incorrectly modified code without clues. We present WizardMerge, an auxiliary tool that leverages merging results from Git to retrieve code block dependency on text and LLVM-IR level and provide suggestions for developers to resolve errors introduced by textual merging. Through the evaluation, we subjected WizardMerge to testing on 227 conflicts within five large-scale projects. The outcomes demonstrate that WizardMerge diminishes conflict merging time costs, achieving a 23.85% reduction. Beyond addressing conflicts, WizardMerge provides merging suggestions for over 70% of the code blocks potentially affected by the conflicts. Notably, WizardMerge exhibits the capability to identify conflict-unrelated code blocks that require manual intervention yet are harmfully applied by Git during the merging.

CCS Concepts: • **Software and its engineering** → **Software configuration management and version control systems**; **Auto-mated static analysis**; • **Theory of computation** → **Program analysis**.

Additional Key Words and Phrases: Version control, Code merging, Static analysis

**ACM** **Reference** **Format:**

Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian. 2024. WizardMerge - Save Us From Merging Without Any Clues. 1, 1 (July 2024), [22](#_bookmark57) pages. [https://doi.org/10.1145/nnnnnnn.nnnnnnn](https://doi.org/10.1145/nnnnnnn.nnnnnnn)

#### 1      Introduction

As the most globally renowned version control system, Git [[17](#_bookmark41)] has been helping with the development and maintenance of millions of projects. To automatically merge the codes from two different versions, Git utilizes a Three-way Merge Algorithm [[42](#_bookmark67)] as the default strategy on the text level. The textual-based merging is general and suitable for various

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image002.gif)|

  

∗Corresponding author.

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image003.gif)|

  

Authors’ Contact Information: Qingyu Zhang, The University of Hong Kong, Hong Kong, China, [z1anqy@connect.hku.hk;](mailto:z1anqy@connect.hku.hk) Junzhe Li, The University of Hong Kong, Hong Kong, China, [jzzzli@connect.hku.hk;](mailto:jzzzli@connect.hku.hk) Jiayi Lin, The University of Hong Kong, Hong Kong, China, [linjy01@connect.hku.hk;](mailto:linjy01@connect.hku.hk) Jie Ding, The University of Hong Kong, Hong Kong, China, [dingjie999888@icloud.com;](mailto:dingjie999888@icloud.com) Lanteng Lin, Beijing University of Posts and Telecommunications, Beijing, China, [lantern@bupt.edu.cn;](mailto:lantern@bupt.edu.cn) Chenxiong Qian, The University of Hong Kong, Hong Kong, China, [cqian@cs.hku.hk.](mailto:cqian@cs.hku.hk)

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image003.gif)|

  

Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components of this work owned by others than the author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from [permissions@acm.org.](mailto:permissions@acm.org)

© 2024 Copyright held by the owner/author(s). Publication rights licensed to ACM. Manuscript submitted to ACM

Manuscript submitted to ACM                                                                                                                                                                                                                                   1

  

types of text files and the merging tempo is proved to be the fastest. However, it simply considers lines of code, neglecting all syntax and semantic information, which leads to potential bugs in the merged version of the project [[22](#_bookmark46), [36](#_bookmark61)].

To mitigate the shortcomings of traditional textual-based merging, multiple merging approaches [[4](#_bookmark28), [5](#_bookmark29), [23](#_bookmark47), [34](#_bookmark58), [35](#_bookmark60), [44](#_bookmark69), [45](#_bookmark70)] are proposed for enhancing the correctness of merging results. These tools can be classified as **Structured** **Merging** and **Semi-structured** **Merging**. Structured merging tools [[23](#_bookmark47), [35](#_bookmark60), [44](#_bookmark69), [45](#_bookmark70)] transform the code intended for merging into an Abstract Syntax Tree (AST), thereby converting the code merging into the merging of edges and nodes on the AST [[1](#_bookmark25)]. Semi-structured merging [[4](#_bookmark28), [5](#_bookmark29), [34](#_bookmark58)], akin to structured merging, designates certain semantically insensitive AST nodes (e.g., strings) as text, allowing them to be directly processed by the text-based merging strategy during the

merging process [[8](#_bookmark32)].

Despite structured and semi-structured merging tools facilitating code merging, challenges arise when it is unclear which version of the code should take precedence. This uncertainty leads to merging conflicts. In the context of large-scale software projects, conflicts can become a source of frustration for developers, as they must dedicate significant effort to resolve the issues that arise from merging before they can release the final merged version [[2](#_bookmark26), [32](#_bookmark56)]. Addressing these conflict resolutions has given rise to merging assistance systems. These systems are designed with objectives such as selecting the most suitable developers to resolve conflicts [[9](#_bookmark33)], prioritizing the resolution order for conflicts [[33](#_bookmark59)], and even automatically generating alternative conflict resolutions [[44](#_bookmark69)]. With the rapid advancements in machine learning technologies in recent years, researchers have turned to machine learning approaches [[3](#_bookmark27), [11](#_bookmark35)–[13](#_bookmark37), [15](#_bookmark39), [38](#_bookmark63), [43](#_bookmark68)]. These systems are trained using extensive datasets containing merge conflict scenarios and can provide predictions on the most suitable resolution strategy for each conflicting portion of code.

While these researches have made noteworthy contributions to code merging, they still exhibit the following limitations. Firstly, structured [[4](#_bookmark28), [23](#_bookmark47), [35](#_bookmark60), [44](#_bookmark69), [45](#_bookmark70)] and semi-structured [[5](#_bookmark29), [34](#_bookmark58)] still generate conflicts when the direct merging between two AST noes failed, and cannot assist developers in resolving them. Secondly, conflict resolution assistance systems [[9](#_bookmark33), [33](#_bookmark59), [44](#_bookmark69)] focus solely on resolving conflicts, overlooking potential errors introduced by incorrectly applied but conflict-free code. Though machine learning approaches [[3](#_bookmark27), [11](#_bookmark35), [12](#_bookmark36), [15](#_bookmark39), [38](#_bookmark63), [43](#_bookmark68)] can automate conflict merging, the functionality of the resulting code may not ensure alignment with developers’ expectations. Additionally, machine learning methods necessitate specific code datasets for pre-training, which raises universality issues when applied to more diverse code projects. Moreover, the developers often have unpredictable operations on resolving conflicts, where the auto-generated conflict resolution may fail to meet the expectations of them.

To address these limitations, we introduce a novel approach aiming to aid developers in handling complex code-merging scenarios. Recognizing that **developers have unpredictable ultimate needs**, our approach focuses on grouping conflicts by relevance and providing prioritized suggestions for resolving conflicts rather than resolving them directly. Our approach also detects the incorrectly applied codes as they can be easily neglected but cumbersome to handle for developers. To realize these design goals, our approach requires a thorough understanding of the dependency relations in the two merging candidate versions. Firstly, we perform definition dependency graph construction on LLVM Intermediate Representation (IR) [[28](#_bookmark52)] and utilize definition indicators extracted from Clang [[24](#_bookmark48)] to indicate the definition name and ranges in the source file. Following graph construction, our approach analyzes the Git merge outcomes that present all the conflicts and modified code blocks. Then, we align the definitions in the source file with code snippets in Git’s outcomes to map from LLVM-IR to the Git merge results. Given the dependency graph analysis and aligned Git merge results, we further conduct an edge-sensitive algorithm to detect incorrectly applied codes, which we deem as conflicts since they require developers’ attention as well. Finally, we assign priorities to these conflicted nodes and produce merging suggestions.

  

We implement a prototype WizardMerge for C/C++ projects as most large-scale projects are written in C/C++ [[20](#_bookmark44)]. We evaluate WizardMerge on 227 conflicts from five popular large projects. The outcomes demonstrate that Wiz-ardMerge diminishes conflict merging time costs. Beyond addressing conflicts, WizardMerge provides merging suggestions for most of the code blocks affected by the conflicts. Moreover, WizardMerge exhibits the capability to identify conflict-unrelated code blocks which should require manual intervention yet automatically applied by Git.

In summary, we make the following contributions:

• We proposed a novel approach to infer the dependency between text-level code blocks with modification.

• We proposed a remarkable methodology, which aims to pinpoint code blocks that have been automatically applied by Git but introduce unexpected results.

• We implemented a code merging auxiliary prototype named WizardMerge based on the approaches and evaluated the system on projects with large-scale code bases. We will open source WizardMerge and evaluation datasets at [https://github.com/aa/bb.](https://github.com/aa/bb)

#### 2      Background

**2.1**      ![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image004.gif)|   |
|---|
|\|   \|<br>\|---\|<br>\|V2 = Stmt2(V1);\|| |   |
|---|
|\|   \|<br>\|---\|<br>\|V1 = Stmt1();\|| **Git Merging**

|   |
|---|
||
||![Text Box: Base](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image005.gif)|

  

|   |
|---|
||
||![Text Box: V4 = Stmt2(V2);](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image006.gif)|

  

![Text Box: V5 = Stmt4(V2, V3);](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image007.gif)

![Text Box: V6 = Stmt5(V1);](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image008.gif)

  

𝐶1            𝐶2

  

|   |
|---|
|\|   \|<br>\|---\|<br>\|V3 = Stmt3(V1, V2);\|| 𝐶3 𝑽𝒂𝒓𝒊𝒂𝒏𝒕 𝑨

  

**_×_**                                          **_×_**

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image017.gif)![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image023.gif)![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image025.gif)![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image026.gif)|   |
|---|
|\|   \|<br>\|---\|<br>\|V7 = Stmt6(V6);\||  |   |
|---|
|\|   \|<br>\|---\|<br>\|\\|   \\|   \\|   \\|   \\|<br>\\|---\\|---\\|---\\|---\\|<br>\\|**_Variant_** **_A_** **_（_****_VA_****_）_**\\|<br>\\|**_Line 1_**<br><br>**_Line 2_**<br><br>**_Line 3_**\\|V2 = Stmt1();\\|**_DCB 1_**\\|<br>\\|V1 = Stmt2(V2);\\|<br>\\|V4 = Stmt2(V2);\\|<br>\\|**_Line 4_**\\|V3 = Stmt3(V1, V2);\\|<br>\\|**_Line 5_**\\|V5 = Stmt4(V2, V3);\\|<br>\\|V6 = Stmt5(V1);\\|<br>\\|**_Line 6_**\\|<br>\\|**_Line 7_**\\|V7 = Stmt7(V5, V6);\\|**_DCB 3_**\\||    |   |
|---|
|\|   \|<br>\|---\|<br>\|\\|   \\|   \\|<br>\\|---\\|---\\|<br>\\|**_DCB 2_**\\|**_Variant B_** **_（_****_VB_****_）_**\\|<br>\\|V1 = Stmt1();\\|<br>\\|V2 = Stmt2(V1);\\|<br>\\|V3 = Stmt3(V1, V2);\\|<br>\\|V5 = Stmt4(V2, V3);\\|<br>\\|V6 = Stmt5(V1);\\|<br>\\|**_DCB 4_**\\|V7 = Stmt2(V5);\\|<br>\\||    |   |
|---|
|\|   \|<br>\|---\|<br>\|\\|   \\|<br>\\|---\\|<br>\\|**_Merged Commit_**\\|<br>\\|V2 = Stmt1();\\|<br>\\|V1 = Stmt2(V2);\\|<br>\\|V4 = Stmt2(V2);\\|<br>\\|V3 = Stmt3(V1, V2);\\|<br>\\|V5 = Stmt4(V2, V3);\\|<br>\\|V6 = Stmt5(V1);\\|<br>\\|**_<Conflict>_**\\||  𝐶6

  

𝑩𝒂𝒔𝒆

  

𝐶4

  

𝑴𝒆𝒓𝒈𝒆𝒅

𝑪𝒐𝒎𝒎𝒊𝒕

𝐶5 𝑽𝒂𝒓𝒊𝒂𝒏𝒕 𝑩

Fig. 1. Three-way merging workflow of Git.

  

**_Developer_**

Git is a free and open-source distributed version control system designed for efficient handling of projects of all sizes [[17](#_bookmark41)]. Git merge, as a key operation, facilitates the integration of changes between versions and is often used to consolidate code changes, bug fixes, or new features [[18](#_bookmark42)]. Git supports multiple textual-based merging algorithms [[7](#_bookmark31)]. Take the default and most commonly employed algorithm, Three-way Merging Algorithm [[42](#_bookmark67)] as an example, the workflow is shown in [Figure 1](#_bookmark1). When the developer tries to merge two commits (i.e., 𝐶3 **Variant A** **(VA)** and 𝐶5 **Vairant** **B (VB)** in [Figure 1](#_bookmark1)), Git first traverses the commit-graph back until it reaches the nearest common ancestor commit (i.e.,

𝐶2, named **Base**). Then, all source code blocks of each newer version are mapped to the corresponding code blocks in base versions, and the modified code blocks of either VA or VB are regarded as **Diff** **Code** **Blocks** **(DCBs)**. A DCB in a

  

variant may contain zero or multiple lines of code, indicating the removal of an existing line or modification/addition of multiple continuous lines. After the code matching stage, Git will iterate each code block tuple, and determine which commit should be applied according to the following rules:

• If code blocks of _VA_ and _VB_ remain the same as the code block of _Base_, or both of them are changed in the same way, inherit the code block from _VA_. In [Figure 1](#_bookmark1), both VA and VB remove the definition of _v4_ (i.e., deleting line 4 of Base). Due to identical modifications at the same locations, Git applies these changes. It is noteworthy that even for code removal, VA and VB maintain a pair of empty DCBs, symbolizing the deletion of lines.

• If only one code block of either _VA_ or _VB_ is changed, Git applies this code block, and marks this code block and the corresponding code block in the other variant as DCBs. In [Figure 1](#_bookmark1), VA alters lines 1-2 and introduces a new line at

3. Git’s code-matching algorithm identifies these modifications, designating lines 1-3 of VA as _DCB_ _1_ and lines 1-2 of VB as _DCB 2_. Since only _DCB 1_ changes, Git applies it to generate the code for the Merged Commit at lines 1-3.

• If both code blocks from _VA_ and _VB_ are changed, and the modifications are different, Git marks both code blocks as DCBs. As Git cannot decide which one should be applied, it emits a conflict containing the two DCBs. In [Figure 1](#_bookmark1), VA and VB independently modify the definition of _v7_. Git detects these alterations, marking line 7 of VA as _DCB_ _3_ and line 6 of VB as _DCB_ _4_. Since these two modifications differ, Git cannot determine which one to apply. Consequently, Git signals the inconsistency between _DCB_ _3_ and _DCB_ _4_, prompting developers to resolve this conflict.

# ![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image032.gif)VA                                                                                 Base

**_VB_**                                                                                 **_Merged_**

Fig. 2. An example of a clean but incorrect Git merge with the three-way merging algorithm

#### 2.2      Limitations of Existing Approaches

Despite being the most widely used code merging tool currently, Git merge faces the following two challenges: _(Challenge_ _1)_ _It_ _cannot_ _ascertain_ _developers’_ _expectations_ _for_ _code_ _functionality._ Consequently, when both versions modify the same code, Git treats this discrepancy as a conflict, necessitating manual intervention for resolution. We discussed how the conflicts are generated in §[2.1](#_bookmark0). _(Challenge 2) It cannot ensure the correctness of successfully applied and conflict-free code_ _blocks._ Even when Git successfully merges two versions without conflicts, the merged code may be generated with new bugs. [Figure 2](#_bookmark2) shows an incorrect merging scenario. Only the first line of _VA_ and the third line of _VB_ are changed.

  

Using Git merge, the two versions of code can be easily merged without any conflict (i.e., the first line applied from VA and the second line applied from VB). Unfortunately, in the merged version, the definition of variable _a_ has been removed, which causes a compilation error at the third line. These DCBs enforced by Git without conflict, yet resulting in unexpected results, are termed **Violated DCBs**. Given that these violated DCBs are not flagged as conflicts and are automatically incorporated by Git merge, they tend to introduce concealed programming errors into the merged code. In the most severe scenarios, these error-prone code segments might seemingly execute successfully but could potentially give rise to security issues post-deployment [[22](#_bookmark46), [36](#_bookmark61)].

Over the years, researchers have concentrated on enhancing the quality of merged code and conflict resolution. However, these two basic limitations have not yet been fully resolved. On the one hand, structured [[23](#_bookmark47), [35](#_bookmark60), [44](#_bookmark69), [45](#_bookmark70)] and semi-structured [[4](#_bookmark28), [5](#_bookmark29), [34](#_bookmark58)] merging still report conflicts when the same AST node is modified simultaneously by two merging candidate branches. On the other hand, the conflict resolution assistance system [[33](#_bookmark59), [44](#_bookmark69)] and machine learning approaches [[3](#_bookmark27), [11](#_bookmark35)–[13](#_bookmark37), [15](#_bookmark39), [38](#_bookmark63), [43](#_bookmark68)] only consider conflicts reported by the merge tool, ignoring all code snippets that have been applied. Additionally, automatic software merging tools [[3](#_bookmark27)–[5](#_bookmark29), [11](#_bookmark35)–[13](#_bookmark37), [15](#_bookmark39), [23](#_bookmark47), [34](#_bookmark58), [35](#_bookmark60), [38](#_bookmark63), [43](#_bookmark68)–[45](#_bookmark70)] neglect an essential observation: **developers** **have** **unpredictable** **ultimate** **needs**, therefore cannot fully overcome _Challenge_ _1_. To better illustrate this, we provide an example of implementing the Fibonacci sequence in [Figure 3](#_bookmark4). Two versions, A and B, calculate the Fibonacci sequence using recursion and iteration, respectively. When attempting to merge these versions, a conflict arises, and two developers resolve it. Although both developers decide to use the iterative approach from version B to avoid stack overflow for large values of 𝑛, they implement different strategies for handling negative

values of 𝑛. Developer A, inspired by version A, uses an unsigned definition for 𝑛 to eliminate negative values (line 1

in resolution A). In contrast, Developer B opts to return zero when 𝑛 is less than zero (line 3 in resolution B). Since developers’ decisions on conflict resolution are unpredictable, automatic software merging tools often fail to produce an acceptable resolution in such cases.

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image034.gif)|

  

Fig. 3. An example of different resolutions to the same conflict

Furthermore, machine learning systems [[3](#_bookmark27), [11](#_bookmark35)–[13](#_bookmark37), [15](#_bookmark39), [38](#_bookmark63), [43](#_bookmark68)] are limited in their ability to resolve conflicts in large codebases due to input and output sequence constraints. However, these conflicts typically do not significantly impact

  

developers’ time. For example, even MergeGen [[13](#_bookmark37)], the most advanced system available, failed to achieve satisfactory results when applied to the large-scale targets evaluated in §[4](#_bookmark17). Upon completing MergeGen’s pre-training, we discovered that it produced no correct outputs, achieving an accuracy of 0%. This failure can be attributed to MergeGen’s default input and output sequence length limitations of 500 and 200 BPE, respectively [[13](#_bookmark37)]. These limitations are excessively stringent for large-scale project source code, leading to truncation of code beyond the prescribed lengths. However, the end of the truncated code is usually far away from where conflict occurs and MergeGen cannot master the ability to resolve the conflicts given the trimmed training and testing sets. Attempts to relax these length limitations are impractical due to increased GPU memory consumption during training. Moreover, while machine learning techniques excel in generating merge results, they may not always align perfectly with developers’ expectations. Although MergeGen’s evaluation results demonstrate superior conflict code matching precision and accuracy on a line of code basis compared to previous works [[13](#_bookmark37)], it still falls short of expectations in some cases, with average precision and accuracy rates of 71.83% and 69.93%, respectively.

#### 3      Design

Given the challenges outlined in the preceding section, we have identified two key insights. Addressing _Challenge 1_, we recognize that the expected outcomes of merges are subjective and open-ended. Consequently, our approach emphasizes guiding developers in resolving conflicts rather than prescribing resolutions. To tackle _Challenge_ _2_, it is imperative to analyze all modified code. To this end, we present WizardMerge, a novel conflict resolution assistance system, which provides developers with clues and aids in conflict resolution. Moreover, WizardMerge emphasizes all modified code snippets, not just conflicts, enabling it to uncover hidden discrepancies and preempt potential errors. A detailed design of WizardMerge will be presented in the subsequent subsections.

#### 3.1      Overview

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image060.gif)|

  

|   |
|---|
|\|   \|<br>\|---\|<br>\|…\|| Manuscript submitted to ACM

  

Fig. 4. System overview of WizardMerge.

  

[Figure 4](#_bookmark5) depicts the system overview of WizardMerge. WizardMerge first consumes compilation metadata from the two merge candidates through LLVM[[28](#_bookmark52)]. WizardMerge employs LLVM based on two insights: i) For large projects, code is compiled before a new version release to ensure compilation success, allowing WizardMerge to collect metadata without introducing additional merging steps; ii) LLVM Intermediate Representation (IR) provides a robust static analysis framework and comprehensive debug information, facilitating WizardMerge in performing definition dependency reasoning. To bridge LLVM IR analysis with merge results from Git, the compilation metadata also encompasses definition indicators. A definition indicator serves as the representation of a named definition section at the source code level, indicating the range and name of a definition. For example, a definition indicator of a function records its function name, along with its start and end line numbers in the source code file. Subsequently, To identify relationships between all modified codes, WizardMerge advances to the **Dependency Graph Generation** stage. Instead of focusing only on conflicts, dependencies for all definitions are analyzed. WizardMerge creates vertices for the definitions and establishes edges between vertices according to the dependency relationships of definitions analyzed from LLVM IRs. Following this, the **Overall** **Dependency** **Graph** **(ODG)** is generated to represent the dependencies among all the definitions in both candidate versions of the target project. To attach Git’s textual results with IR-level dependency information, WizardMerge aligns Git merge’s results and ODG to match DCBs with vertices in ODG through **Graph-Text** **Alignment**. This stage is capable of mapping DCBs to the corresponding definition nodes in each ODG, refining definition nodes into DCB nodes, and transforming definition dependencies into DCB dependencies. To rectify errors introduced by Git merge, WizardMerge employs an edge-sensitive algorithm in the **Violated** **DCB** **Detection** stage, detecting both conflict-related and conflict-unrelated violated DCBs. These violated DCBs and conflicts are categorized, prioritized based on the dependency graph, and presented as suggestions to developers to assist in resolution.

#### 3.2      Dependency Graph Generation

WizardMerge first constructs ODG based on the LLVM IR and definition indicator. WizardMerge currently supports the following types of nodes and edges:

_Graph node._ Subsequently, WizardMerge generates nodes for each named definition with the corresponding indicator. Considering the types of definitions, WizardMerge creates three types of nodes based on the indicator information: **Type** **Node** **(TN)** represents composite types, e.g. structure, class, type alias, and enumeration; **Global** **Variable** **Node** **(GN)** represents global variable; **Function** **Node** **(FN)** represents function.

_Graph edge._ Each named definition may depend on other definitions within the project. For instance, a function might utilize a global variable or return a pointer with a specific structure type. In the ODG, these usage dependencies among definitions are deduced from LLVM IRs and represented by the edges linking one node to another. The types of edges are defined by the types of their two connected nodes. An edge of type A-B represents a dependency from node A to node B. In WizardMerge, five classifications of edges are supported: TN-TN, GN-TN, FN-TN, FN-GN and FN-FN. Note that WizardMerge will never generate an edge from TN to FN, even though member functions may be defined within a composite type. Instead, WizardMerge creates TN for all functions, irrespective of whether the function is standalone or nested within another scope. If an FN represents a nested function defined inside a composite type, an

FN-TN edge is built to represent the dependency from the function to the specific type.

#### 3.3      Resolving Suggestion Generation

In this stage, WizardMerge extracts DCB information from Git merge to aid in the analysis. Then, merging suggestions can be generated based on the analysis results.

  

_3.3.1_      _Graph-Text_ _Alignment._ The merge results cannot be directly linked to the corresponding node in ODG. This is primarily because these results are raw texts that provide no additional clues for program analysis. In the raw results, a single large DCB may cover multiple nodes and a single node may encompass several DCBs. This N-to-N relationship complicates the analysis of the graph.

### 

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image082.gif)|

  
Before Graph-Text Alignment               After Graph-Text Alignment

Fig. 5. Example alignment between node and DCB text.

WizardMerge initially associates each DCB with its corresponding definition indicators. To identify all nodes contained in the file where the DCB is situated, WizardMerge extracts the nodes whose range intersects with the DCB range. If no matched node is found, it indicates that the DCB is not in a definition (e.g., newlines or comments) and will not be further analyzed by WizardMerge. Treating each DCB as a "line segment", its start and end line numbers represent the coordinates of the line segment’s two endpoints on the coordinate axis. Similarly, each definition indicator of a node is considered as a unique "color". The range of each node signifies a specific coordinate interval that is colored with the definition represented by the node. Thus, retrieving the node during each DCB traversal can be seen as a color query for the interval [𝑠𝑡𝑎𝑟𝑡_𝑙𝑖𝑛𝑒, 𝑒𝑛𝑑_𝑙𝑖𝑛𝑒], and interval modification and information maintenance can be efficiently managed using line segment trees. For a target project containing 𝑁 lines of code, each "coloring" and "color query" operation can be achieved with a time complexity of 𝑂 (𝑙𝑜𝑔2𝑁 ) [[10](#_bookmark34)]. WizardMerge subsequently logs all DCBs corresponding to each node, organizing them in ascending order of line numbers. The node is then subdivided into finer-grained nodes. The ranges of these finer-grained nodes are updated concurrently based on the original node and their corresponding unique DCBs. Fine-grained nodes belonging to the same definition will be connected sequentially in range order, which implicitly expresses the dependency between code blocks. This mechanism ensures that each node in ODG aligns with at most one DCB, effectively preventing any matching confusion within the ODG.

Taking [Figure 5](#_bookmark7) as an instance, there are two definitions: _Named_ _Def_ _1_, _Named_ _Def_ _2_. Before alignment, the nodes

are created with their range information. Let’s consider two DCBs associated with these definitions ([𝐿2, 𝐿5] and [𝐿6, 𝐿7]). To establish a one-to-one mapping between DCBs and nodes, each node can be divided into multiple ones, while unaffected lines are excluded as they do not influence the merging. For DCB [𝐿2, 𝐿5], WizardMerge detects that [𝐿2, 𝐿3] belongs to _Named_ _Def_ _1_ and [𝐿5, 𝐿5] belongs to _Named_ _Def_ _2_. Consequently, a new node starting from 𝐿2 to 𝐿3 will replace the original node for _Named_ _Def_ _1_. As [𝐿1, 𝐿1] remains unchanged, it will be discarded. Furthermore, a new node from 𝐿5 to 𝐿5 is also formed since [𝐿6, 𝐿7], representing another portion of _Named_ _Def_ _2_, corresponds to the DCB [𝐿6, 𝐿7]. To indicate the code dependency from lines to previous lines within the same definition, an additional edge from node [𝐿6, 𝐿7] to node [𝐿5, 𝐿5] is established.

As WizardMerge is tasked with managing two code versions (VA and VB), some nodes in the ODG of VA must possess a corresponding mirror node in the ODG of VB. WizardMerge also constructs a mapping of each node to its

  

matched node in the other version’s ODG by matching the name of the definition and the DCB metadata linked to the node. Note that if one node is aligned with a DCB, it must have one unique mirror node with the other DCB in the same DCB pair. If the node has a matched node, the matched node must be the mirror node and the definition name is consistent with the current node. The matched node results contribute to the violated DCB detection via dependency missing determination. Besides, the ODG’s size can also be shrunk if there are nodes not attached to any DCBs, which means they are not changed during merging. After the node deletion, the newly generated graph is called **Shrunk Dependency Graph (SDG)**.

_3.3.2_      _Violated_ _DCB_ _Detection._ Violated DCBs are the underlying cause of seemingly successful but erroneous merging. To offer developers a comprehensive understanding of the merging process, WizardMerge detects violated DCBs based on an edge-sensitive algorithm and deems them as conflicts requiring resolution suggestions. Given the SDGs and node matching relationship, WizardMerge traverses each edge in the two versions of the SDG separately. Each edge, starting from 𝑣 and ending at 𝑢, signifies that 𝑣 relies on 𝑢. WizardMerge determines which version of the DCB associated with 𝑢 and 𝑣 is ultimately applied in the merge results (i.e., from VA, VB, or conflict). WizardMerge then ascertains whether these nodes represent violated DCBs by the applied versions and matching nodes of 𝑢 and 𝑣. We introduce all the safe (i.e. not violated DCBs detected) and violated scenarios as follows.

_Symbols Definition._ Let the SDGs from VA and VB be represented as 𝐺𝐴 = _<_ 𝑉 (𝐺𝐴), 𝐸 (𝐺𝐴) _>_ and 𝐺𝐵 = _<_

𝑉 (𝐺𝐵 ), 𝐸 (𝐺𝐵 ) _>_, where 𝑉 indicates the vertices set and 𝐸 indicates the edges set. For each vertex 𝑣 ∈ 𝑉 , we represent its mirror node as 𝑀𝑖 (𝑣) and use 𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑣)) to indicate whether the mirror node of 𝑣 is a matching node (𝑇𝑟𝑢𝑒 or

𝐹𝑎𝑙𝑠𝑒). 𝑣 belongs to only one of the 𝑉𝑁 , 𝑉𝐴, and 𝑉𝐶 vertex sets, representing not applied, applied, and conflict node sets

respectively. For each edge 𝑒 (𝑣, 𝑢) ∈ 𝐸, it represents an edge from 𝑣 to 𝑢, indicating 𝑣’s dependency on 𝑢. According to the construction principles of SDG, there are two basic facts:

•  If 𝑣 ∈ 𝑉𝐴, then 𝑀𝑖 (𝑣) ∈ 𝑉𝑁 , and vice versa.

•  If 𝑣 ∈ 𝑉𝐶 , then 𝑀𝑖 (𝑣) ∈ 𝑉𝐶 .

As each 𝑣 ∈ 𝑉𝐴 must indicate another vertex (i.e., 𝑀𝑖 (𝑣)) belongs to 𝑉𝑁 , WizardMerge only analyzes the cases where

𝑣 ∈ 𝑉𝐴 or 𝑣 ∈ 𝑉𝐶 to avoid duplicate traversal.

_Safe_ _Scenarios._ For each 𝑣 and an edge 𝑒 = (𝑣, 𝑢), we define the scenarios satisfying any of the following four formulas as safe:

|   |   |
|---|---|
|𝑣 ∈ 𝑉𝐴, 𝑢 ∈ 𝑉𝐴|(1)|
|𝑣 ∈ 𝑉𝐴, 𝑢 ∉ 𝑉𝐴, 𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑢)) = 𝑇𝑟𝑢𝑒|(2)|
|𝑣 ∈ 𝑉𝐶, 𝑢 ∈ 𝑉𝐴|(3)|
|𝑣 ∈ 𝑉𝐶, 𝑢 ∉ 𝑉𝐴, 𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑢)) = 𝑇𝑟𝑢𝑒|(4)|

In SDGs, the above formulas can be regarded as an edge-sensitive detection algorithm, which is visiualized as depicted in [Figure 6](#_bookmark13). For [Equation 1](#_bookmark9), if both 𝑢 and 𝑣 are applied, it means the dependency relation between nodes within the same SDG (as shown in _Scenario_ _1_ of [Figure 6](#_bookmark13)). By the same token, _Scenario_ _3_ also illustrates such a direct dependency

represented by [Equation 3](#_bookmark11). For [Equation 2](#_bookmark10) (_Scenario 2_ in [Figure 6](#_bookmark13)), 𝑣 is applied but 𝑢 is either not applied (𝑢 ∈ 𝑉𝑁 ) or in conflict status (𝑢 ∈ 𝑉𝐶 ). Nevertheless, if the mirror of 𝑢 matches 𝑢 (𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑢)) = 𝑇𝑟𝑢𝑒), it means that at least in the other SDG, 𝑣 can find its corresponding dependency applied (i.e., 𝑀𝑖 (𝑢)) although 𝑢 is not applied, or both versions of conflicted 𝑢 hold the same dependency for 𝑣. To this end, WizardMerge uses a virtual dependency edge from 𝑣 to

  

𝑀𝑖 (𝑢) to indicate a cross dependency between nodes from different SDGs. Similarly, [Equation 4](#_bookmark12) (_Scenario_ _4_) is also regarded as safe with the same reason.

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image084.gif)|

  

Fig. 6. Edge traversal scenarios visualization

_Violated_ _Scenarios._ For each 𝑣 and an edge 𝑒 = (𝑣, 𝑢), all the scenarios not detected as safe are regarded as violated scenarios. These scenarios are:

𝑣 ∈ 𝑉𝐴, 𝑢 ∉ 𝑉𝐴, 𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑢)) = 𝐹𝑎𝑙𝑠𝑒                                                              (5)

𝑣 ∈ 𝑉𝐶, 𝑢 ∉ 𝑉𝐴, 𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑢)) = 𝐹𝑎𝑙𝑠𝑒                                                              (6)

The _Scenario_ _5_ and _Scenario_ _6_ in [Figure 6](#_bookmark13) visualize [Equation 5](#_bookmark14) and [Equation 6](#_bookmark15) respectively. Once 𝑢 is not applied or in conflict, and the mirror node of 𝑢 does not match it (𝑚𝑎𝑡𝑐ℎ(𝑀𝑖 (𝑢)) = 𝐹𝑎𝑙𝑠𝑒), it means that 𝑣 misses dependency from 𝑢 in both SDGs (when 𝑢 ∈ 𝑉𝑁 ), or 𝑀𝑖 (𝑢) holds a different dependency from 𝑢, which does not match 𝑣’s requirement (when 𝑢 ∈ 𝑉𝐶 ).

WizardMerge identifies these cases are incorrectly handled by Git merge and marks the corresponding 𝑣, 𝑢, 𝑀𝑖 (𝑣)

and 𝑀𝑖 (𝑢) as violated. Note that vertices with conflicts are always treated as violated, regardless of whether they are encountered in safe or violated scenarios. Finally, the DCBs attached to these vertices are also marked as violated.

  

![](data:image/png;base64,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)

Fig. 7. MDG example. Each node in MDG contains exactly a pair of mirror nodes in SDG. Each edge is built if there is at least one SDG edge from one SDG node to another.

_3.3.3_      _Priority-Oriented_ _Classification._ WizardMerge further minimizes the number of nodes that require processing and constructs a **Minimum** **Dependency** **Graph** **(MDG)** to emulate the dependencies of the merged version of code. In the MDG, each node will consist of either an applied node or a conflict node and its matching node, and the edges between nodes will be created based on various SDGs according to the applying status. Specifically, while establishing the MDG, WizardMerge will traverse each conflict node or node associated with violated DCBs, and based on the applying status of the node, it will switch to the corresponding version of SDG and continue traversing the nodes in that particular SDG. Particularly, if the node is in conflict status, then the outcoming edges of both the mirror of the node and itself would be iterated. Concurrently, WizardMerge sets up a directed edge between corresponding nodes for the MDG, relying on the encountered edges during traversal. Ultimately, the MDG encapsulates the essential dependencies of the two versions of code to be merged.

[Figure 7](#_bookmark16) shows an instance illustrating how MDG is built from SDG. Each MDG node is created from a pair of nodes related to conflict or violated DCBs. SDG node _A-E_ are from VA and all _Mirror_ nodes are from VB. Starting from A and its mirror, WizardMerge iterates the outcoming edges and finds B is linked to a violated DCB. Then, WizardMerge creates a new node _MDG_Node_B_ containing B and its mirror. As B is not applied while its mirror node is applied, WizardMerge iterates the outcoming edges of the mirror of B and finds a pair of conflict nodes (i.e., E and Mirror), which forms _MDG_Node_E_. WizardMerge then iterates the outcoming edges of both B and Mirror because of the conflict. By the same token, _MDG_Node_D_ and _MDG_Node_C_ are created.

WizardMerge then employs the Tarjan algorithm [[39](#_bookmark64)] to condense the MDG nodes and eliminate cycles. During this process, nodes in the same strongly connected component are regarded as having identical priority, as these nodes may reference each other within the code. In the case of the directed acyclic MDG, WizardMerge divides the MDG into multiple subgraphs based on connectivity. All nodes within each subgraph are grouped, and each node is assigned a priority through the topological sorting of the subgraph. Nodes that appear later in the topological order signify that other nodes have greater dependencies on them, while the nodes themselves have fewer dependencies. Consequently, these nodes are assigned higher priority, enabling developers to address them earlier. In [Figure 7](#_bookmark16), _MDG_Node_C_,

  

_MDG_Node_D_ and _MDG_Node_E_ are in the same cyclic of MDG and are thus assigned the same resolution priority as they depend on each other. After calculating the topological order, WizardMerge suggests that the DCB resolution priority should be {_MDG_Node_C,_ _MDG_Node_D,_ _MDG_Node_E_}, {_MDG_Node_B_} and {_MDG_Node_A_}.

#### 4      Evaluation

In this section, we evaluate the effectiveness of WizardMerge, aiming to answer the following research questions:

•  **RQ1:** Can WizardMerge help reduce the time consumption of merging conflicts?

•  **RQ2:** Can WizardMerge detect violated DCBs related to conflicts applied by Git?

•  **RQ3:** Can WizardMerge detect violated DCBs unrelated to conflicts applied by Git?

_Dataset_ _Collection._ We choose Firefox [[29](#_bookmark53)], Linux Kernel [[40](#_bookmark65)], MySQL [[31](#_bookmark55)], PHP [[31](#_bookmark55)], and LLVM [[28](#_bookmark52)] as the target projects because of their widespread usage as open-source software. Additionally, they boast extensive code bases and are maintained by multiple developers, resulting in repositories that feature intricate commits and merging scenarios. We collect 22 conflict pairs from Firefox, 43 from Linux Kernel, 17 from LLVM, 46 from PHP, and 99 from MySQL with a committing timeframe from 2021 to 2024. For each pair of conflicts, we only focus on the files in C/C++ format and guarantee they either have at least five conflict regions or violated DCBs detected. On average, a conflict contains

824.33 lines of codes and is related to 6.8 files.

_Evaluate with State-of-the-art (SOTA) Systems._ We initially intended to assess WizardMerge alongside other cutting-edge systems. However, JDIME [[4](#_bookmark28)], Mastery [[45](#_bookmark70)], Spork [[23](#_bookmark47)], SafeMerge [[35](#_bookmark60)], AutoMerge [[44](#_bookmark69)], IntelliMerge [[34](#_bookmark58)] and SoManyConflicts [[33](#_bookmark59)] encountered language incompatibility issues within our datasets. Besides, migrating Wizard-Merge to the languages these systems support is impractical because as an LLVM-based project, WizardMerge requires the full support from the LLVM framework, while the compiler front ends of Java [[37](#_bookmark62)], Typescript [[6](#_bookmark30)], and Javascript [[6](#_bookmark30)] are still under development. Furthermore, FSTMerge[[5](#_bookmark29)] purports scalability contingent upon users implementing the merging engine for specific languages. Unfortunately, its C/C++ parsers are either underdeveloped (i.e., accommodating only basic syntax) or unimplemented, rendering it unsuitable for real-world projects. In addition, as mentioned in § [2.2](#_bookmark3), the machine learning approaches [[3](#_bookmark27), [11](#_bookmark35)–[13](#_bookmark37), [15](#_bookmark39), [38](#_bookmark63), [43](#_bookmark68)] meet the sequence length limitation challenge when facing the large-scale targets we chose.

_Evaluation_ _Environment._ We evaluated WizardMerge on an AMD EPYC 7713P server with 64/128 cores/threads, NVIDIA GeForce RTX 4090 (24GB GPU memory), and 256GB memory, running Ubuntu 20.04 with kernel 5.4.0.

#### 4.1      Effectiveness on Conflict Resolution

To address **RQ1**, we conducted a comparative analysis of the time consumption associated with two distinct approaches (Git merge w/ and w/o WizardMerge) for merging conflict pairs. As WizardMerge only provides suggestions instead of resolutions of conflicts, We have to assign two types of technical staff to the manual merging phase: **skilled** **staffs,** researchers with deep understanding of the project, and **ordinary** **staffs,** researchers having general understanding of the project. Initially, the ordinary staff employed WizardMerge to analyze the two versions and performed the merge utilizing the suggestions provided by WizardMerge. Subsequently, the skilled staff manually merged the two candidate versions using Git merge without any additional assistance. We meticulously recorded the number of conflicts that can be analyzed by WizardMerge and the time taken (with precision to the minute) from the commencement of conflict analysis to their complete resolution.

  

Table 1. Conflict coverage of WizardMerge. **Total** **Num** indicates the total number of conflicts in each dataset. **Analyzed** **Num**

indicates the number of conflicts successfully analyzed by WizardMerge. **Cover** **Rate** indicates the ratio of the above two columns.

|   |   |   |   |
|---|---|---|---|
|**Dataset**|**Analyzed** **Num**|**Total** **Num**|**Cover** **Rate**|
|Firefox|159|160|99.38%|
|Linux Kernel|75|88|85.23%|
|LLVM|71|72|96.61%|
|PHP|236|255|92.55%|
|MySQL|797|964|82.68%|

Before comparing the effectiveness of merging w/ and w/o WizardMerge, we first evaluate its ability to assess conflicts. As shown in [Table 1](#_bookmark18), we calculated the total number of conflicts for each dataset and recorded how many conflicts can be successfully analyzed by WizardMerge. WizardMerge can give suggestions of more than 80% conflicts from the five targets. Notice that some conflicts cannot be handled by WizardMerge, we manually dig the reasons behind the failure. In the design section, we mentioned that WizardMerge’s analysis is based on LLVM IR. However, in textual merging results, some conflicts are caused by the content of comments or preprocessing instructions (e.g., _#include_ and macro definition). Since comments do not belong to the code scope, and the preprocessing instructions will be expanded in the Clang front end, LLVM IR will lose the text information of these conflicts, and finally WizardMerge will not give suggestions related to them.

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image088.gif)>70

![Text Box: Time Consumption (mins)](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image089.gif)70

65

60

55

50

45

40

35

30

25

20

15

10

5

|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|||||||||||||||||||||||||
|||   |   |   |   |   |   |   |   |   |   |   |![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image102.gif)||   |   |![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image103.gif)||   |   |![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image104.gif)|
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image105.gif)||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image106.gif)||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image107.gif)||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image108.gif)||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image109.gif)||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image110.gif)|||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image111.gif)|||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image112.gif)|||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image113.gif)|
||
||
||
||
||
||

  
0

## Conflict Pair ID

Fig. 8. Manual merging time w/ and w/o WizardMerge.

The effectiveness of merging w/ and w/o WizardMerge is quantified by time consumption and is shown in [Figure 8](#_bookmark19). When the time consumption of merging a pair of commits exceeds 70 minutes, we will mark it as "_>_ 70" and stop resolving it. In 135 out of 277 conflict scenarios, WizardMerge contributes to reducing version conflict resolution time, and as the time consumption w/o WizardMerge increases, the assistance ability of WizardMerge also becomes more obvious. Remarkably, for the 205th conflict pair, manual merging w/o WizardMerge costs more than 70 minutes, while with the help of WizardMerge, the merging process can be finished in 35 minutes. This exceptional performance is attributed to the Priority-Oriented Classification module in WizardMerge, which provides categorization and fixes priority recommendations for manually resolved code blocks. Simultaneously, the Violated DCB Detection module

  

in WizardMerge automatically identifies code blocks related to conflicts. This feature simplifies developers’ efforts in analyzing dependencies and avoids the substantial time spent on the manual in-depth exploration of extensive code space required by Git merge. However, WizardMerge may not optimize the time consumption for all data points. In 18 out of 277 conflict cases, using WizardMerge could even increase the burden of conflict resolution. This occurs when only a small number of conflicts are present and few code blocks are affected, rendering WizardMerge’s assistance not substantial. Additionally, as mentioned above, WizardMerge fails to generate suggestions on comments or preprocessing instructions.

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image114.gif)Listing 1. Case study for conflict & violated DCB classification and WizardMerge-produced resolving order assignment.

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

  

|   |   |   |
|---|---|---|
|51|**@@**|**\|**|
|52|**@@**|**C-****- Scrollbar Data xxx / Scrollbar Data . h 41 -44 , 41 -43**|
|53|**@@**|**\|**|
|54|**@@**|**C-****- Create For Thumb xxx / Scrollbar Data . h 68 -74 , 66 -72**|
|55|**@@**|**\|**|
|56|**@@**|**A-****- Create For Thumb xxx / Scrollbar Data . h 76 -77**|
|57|**@@**|**\|**|
|58|**@@**|**B-****- ( applied ) Create For Thumb xxx / Scrollbar Data . h 74 -75**|

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image115.gif)[Listing 1](#_bookmark20) presents a case study illustrating how WizardMerge classifies the conflicts and violated DCBs and assigns resolving privileges between **VA** [145d3f6](https://github.com/mozilla/gecko-dev/tree/145d3f680f74e0b7e22be9de4943baf4ff0c859b) and **VB** [281943a](https://github.com/mozilla/gecko-dev/tree/281943a7a7564da09bd6076cac0c47a7a62144dc). Initially, both versions introduce distinct modifications to the _ScrollbarData_ constructor, resulting in a merge conflict (lines 8-17). Subsequently, both versions make different alterations to the parameter types of the _CreateForThumb_ method in the _ScrollbarData_ class, leading to another merge conflict (lines 25-41). However, Git’s strategy reports these conflicts directly to the developer without providing additional clues. In contrast, WizardMerge utilizes dependency analysis between definitions to discern that the constructor of ScrollbarData will be called by _CreateForThumb_ (line 42). Consequently, WizardMerge classifies the two conflicts into one group, assigning higher fix priority to lines 8-17 than to lines 25-41.

Additionally, VA and VB present diverse implementations of the function body of the _CreateForThumb_ method. According to the three-way merging algorithm, Git applied the code snippets from VB (lines 45-46) after detecting these two conflicts. Moreover, within the pair of DCBs represented by lines 43-46, although lines 45-46 are applied without conflicts, it cannot ensure the constructor of _ScrollbarData_ receives the correct parameter types when both conflicts are present. This is because, if the developer chooses to apply all VA modifications when resolving these conflicts, the applied lines 45-46 from VB will lack essential parameters, _aThumbMinLength_ and _aThumbIsAsyncDraggabl_, used by _ScrollbarData_ constructor and the types of _aThumbStart_, _aThumbLength_, _aScrollTrackStart_, and _aScrollTrackLength_ will not match the _ScrollbarData_ constructor prototype. Nevertheless, WizardMerge detects lines 45-46 and its mirror DCB (i.e., lines 43-44) through its innovative dependency analysis approach, treating them as violated DCBs. It reports them to developers along with other conflicts. Since lines 43-46 depend not only on lines 8-17 (function call) but also on lines 25-41 (definition of prototype), these two violated DCBs are grouped with the previous two conflicts, with fixed priority coming after the two conflicts. The resolution suggestions provided by WizardMerge are shown in lines 49-58 of [Listing 1](#_bookmark20), where items prefixed with C indicate pairs of DCBs with conflicts, and items prefixed with A or B indicate DCBs from VA or VB considered as violated DCBs.

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image116.gif)

#### 4.2      Violated DCB Detection

To address both **RQ2** and **RQ3**, our initial step involves establishing the ground truth regarding the number of violated DCBs by manually merging two code versions using Git merge. Subsequently, we employ WizardMerge to conduct an analysis and document the identified violated DCBs. As not all code segments within a DCB pertain to violated

  

Table 2. Violated DCB detection records. **CR** **VA** represents the LoC of CR violated DCB in VA, and **CR** **VB** represents the LoC of CR violated DCB in VB. The format 𝑛/𝑚 indicates that there are 𝑚 LoCs in total, of which WizardMerge detects 𝑛. Similarly, **CU** **VA** represents the LoC of CU violated DCB in VA, and **CU** **VB** represents the LoC of CU violated DCB in VB. The format 𝑛/𝑚 indicates that WizardMerge detect 𝑚 LoCs as CU violated DCB, of which 𝑛 are confirmed. **Recall** and **Precision** with † and ‡ stand for VA and VB respectively.

|   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|
|**Dataset**|**CR-****VA**|**CR-****VB**|**CU-****VA**|**CU-****VB**|**Recall**†|**Recall**‡|**Precision**†|**Precision**‡|
|Firefox|151/174|112/123|75/652|98/717|88.30%|91.06%|11.50%|13.67%|
|Linux Ker-nel|107/131|156/192|171/1245|105/905|81.68%|81.25%|13.73%|11.60%|
|LLVM|212/316|16/23|_N/A_|_N/A_|67.09%|69.57%|_N/A_|_N/A_|
|PHP|92/132|49/139|226/324|213/358|69.70%|35.25%|69.75%|59.50%|
|MySQL|12274/13100|3171/3469|1750/4597|1307/2427|93.69%|91.41%|38.07%|53.85%|

DCBs, we utilize **Line** **of** **Code** **(LoC)** for careful quantification of the violated DCBs. These violated DCBs are further categorized into **Conflict-Related (CR)** and **Conflict-Unrelated (CU)** violated DCBs.

_4.2.1_      _Conflict-Related_ _Violated_ _DCBs._ In addition to addressing conflict DCBs, developers should reassess the DCBs automatically applied by Git merge. Resolving one conflict might potentially impact other sets of DCBs. Throughout our evaluation, we recorded the LoC for DCBs linked to conflicts in both versions. This data reflects WizardMerge’s capability in elucidating the correlation between conflicts and violated DCBs.

[Table 2](#_bookmark21) illustrates the CR-violated DCB detection results of WizardMerge. As VA and VB’s codes are different, the LoC of CR-violated DCBs in the same diff pair may vary as well. Thus, we documented the LoC from both VA and VB. According to the evaluation, 167 out of 227 conflict scenarios are confirmed to contain CR-violated DCBs. For most of the targets, WizardMerge successfully detected more than 60% CR-violated DCBs, providing a promising assistive cue for developers in executing manual fixes and facilitating them in exploring the CR-violated DCBs. Noticeably, although MySQL datasets have the most LoC of CR-violated DCBs, WizardMerge’s detection recall even reaches 93.69%. For LLVM and PHP, WizardMerge showed moderate recalls.

We also delved into the reasons behind WizardMerge’s inability to detect CR-violated DCBs in certain datasets. Firstly, the Linux Kernel encountered compilation failure with the "O0" option, a crucial setting for preserving debug information and preventing code optimization. Although "Og" can be used for better debug ability, it is akin to "O1," resulting in the elimination of significant debug information [[14](#_bookmark38)]. For instance, inlined functions are inserted at the invocation location and subsequently removed from the original code. As WizardMerge depends on the collaboration of debug information and LLVM IR, the absence of debug information leads to the failure of node generation and DCB matching. Secondly, some datasets include violated dependencies among local variables within a specific function or macro definition, which WizardMerge’s analysis currently does not support.

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image117.gif)Listing 2. Case study for conflict-related violated DCBs

1

2

3

4

5

6

7

8

  

![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image118.gif)9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

[Listing 2](#_bookmark22) presents an intriguing case study revealing conflict-related violated DCBs detected by WizardMerge between **VA** [5c5e0e8](https://github.com/git/git/tree/5c5e0e81202667f9c052edb99699818363b19129) and **VB** [1acfe2c](https://github.com/git/git/tree/1acfe2c1225899eab5ab724c91b7e1eb2881b9ab). In VA, the definition for _virtnet_rq_dma_ is present, whereas at the same position in VB, _virtnet_interrupt_coalesce_ is defined. Git identifies these versions’ different modifications as a conflict and prompts manual resolution for developers. Subsequently, Git designates other DCBs as not in conflict and applies them based on their consistency with the base version’s DCBs. Eventually, the code snippets from lines 24, 31, and 50-51 (from VA) along with 37-40 (from VB) are applied. Despite the differences between VA and VB in lines 46-51, lines 46-49 from VB are consistent with the base version’s snippets. According to the three-way merging strategy, Git prefers to apply lines 50-51 from VA. In contrast, WizardMerge analyzes all modified code and discovers that even though the conflict is highlighted, some related DCBs have been applied without alerting the developers. For instance, applying lines 37-40 from VB assumes the presence of struct _virtnet_rq_dma_. Yet, if the developer opts for lines 15-17 as the conflict resolution, the structure becomes undefined within that context. Similarly, applying lines 24, 31, and 50-51 from VA assumes the definition of struct _virtnet_interrupt_coalesce_, while they will cause compilation errors if the developer

  

  

Listing 3. The definitions of function _EnsureNSSInitialized-__ChromeOrContent_ are the same in both versions

  

Listing 4. Difference between commits d2956560d539 and a20620423a53 in file xpcom/base/nsCOMPtr.h

  

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5|**// security / manager / ssl / ns NSSComponent . cpp** **bool** **Ensure** **NSSInitialized** **Chrome** **Or Content () {**<br><br>**...**<br><br>**if** **( XRE_Is Parent Process ()) {**<br><br>**// Create ns ISupports * via template class ns COMPtr .**|   |   |1<br><br>2<br><br>3<br><br>4|**diff -- git a/ xpcom / base / ns COMPtr . h b/ xpcom / base / ns COMPtr . h**<br><br>**--****- a/ xpcom / base / ns COMPtr . h**<br><br>**+++ b/ xpcom / base / ns COMPtr . h** **template <class T>**|
|6|||**nsCOMPtr < nsISupports > nss =**|5|**class MOZ_IS_REFPTR ns COMPtr final {**|
|7|||**do_Get** **Service** **(** **PSM_COMPONENT_CONTRACTID** **);**|6|**...**|
|8|||**if** **(! nss ) {**|7|**};**|
|9|||**return false** **;**|8||
|10|||**}**|9|**...**|
|11|||**initialized** **= true ;**|10||
|12|||**return true** **;**|11|**+ template <>**|
|13||**}**||12|**+ class MOZ_IS_REFPTR nsCOMPtr < nsISupports >**|
|14||**...**||13|**+**                                                        **: private ns COMPtr_base {**|
|15|**}**|||14|**+**         **...**|
|||||15|**+ }**|
|||||||

chooses lines 9-13 as the conflict resolution. Consequently, WizardMerge identifies these code snippets as violated DCBs in conjunction with the conflict. Furthermore, WizardMerge assigns higher priority to the conflict, signifying the dependency of other DCBs on the conflict and suggesting developers resolve the conflict before reconsidering these violated DCBs.

_4.2.2_      _Conflict-Unrelated Violated DCBs._ Throughout this evaluation, we document the LoC for DCBs unrelated to conflicts in both variant versions reported by WizardMerge. Following this, we manually inspected the identified DCBs and calculated the number of LoC genuinely violated among them.

The results of WizardMerge’s detection of CU-violated DCBs are also presented in [Table 2](#_bookmark21). As CU-violated DCBs are isolated from conflicts, WizardMerge demonstrates the capability to detect them, even in scenarios where it fails to handle any conflicts. In total, WizardMerge detected CU-violated DCBs in 137 out of 227 conflict scenarios, while no CU-violated DCBs were found in LLVM. These DCBs, lacking necessary dependent definitions and automatically applied by Git merge without developer notification, do not lead to conflicts but can result in compile errors or even latent bugs affecting users at runtime. While WizardMerge showcases its potential in uncovering violated dependency issues, the evaluation also highlights instances of **false positive** CU-violated DCBs. This arises from the lack of function body analysis in WizardMerge. According to WizardMerge’s DCB detection algorithm §[3.3.2](#_bookmark8), the relationship between two DCBs from the same function relies on line numbers, with the latter one forcibly dependent on the former one. If the two DCBs are applied from different versions, WizardMerge marks them as CU-violated DCBs, even if they are unrelated. This conservative algorithm may result in more false positives, but WizardMerge prioritizes its application to prevent overlooking the detection of CU-violated DCBs.

[Listing 3](#_bookmark23) and [Listing 4](#_bookmark24) present another case study where WizardMerge explores conflict-unrelated violated DCBs within Firefox (between versions **VA** [d295656](https://github.com/mozilla/gecko-dev/tree/d2956560d539c54eaf56297a27b139b862ac858f) and **VB** [a206204](https://github.com/mozilla/gecko-dev/tree/a20620423a5363cf7afdac81e062bc687c29366a)), which were overlooked by Git merge. Specifically, in the file _nsNSSComponent.cpp_, both versions define the function _EnsureNSSInitializedChromeOrContent_ with the same contents. In [Listing 3](#_bookmark23) lines 6-7, a pointer to _nsISupports_ is created via the template class _nsCOMPtr_. The definition of _nsCOMPtr_ is found in _nsCOMPtr.h_. However, in addition to the generic template, a template specialization [[41](#_bookmark66)] is also defined for _nsISupports_ in VB, as shown in lines 11-15 in [Listing 4](#_bookmark24). This specialization of _nsCOMPtr_ for _nsISupports_ allows users to employ _nsCOMPtr<nsISupports>_ similarly to how they use _nsISupports*_ and _void*_, essentially as a ’catch-all’ pointer pointing to any valid [XP]COM interface [[30](#_bookmark54)]. When attempting to merge _nsCOMPtr.h_ from the two versions, Git applies the file from VA because the file from VB is identical to the file from the base version. Consequently, in the function _EnsureNSSInitializedChromeOrContent_, the usage of _nsCOMPtr<nsISupports>_ will follow lines 4-7 in

  

|   |
|---|
||
||![](file:///C:/Users/richa/AppData/Local/Temp/msohtmlclip1/01/clip_image119.gif)|

  
[Listing 4](#_bookmark24), causing _nss_ to only be able to point to the single _[XP]COM-correct_ _nsISupports_ instance within an object. This inconsistency can lead to a potential **type** **confusion** bug [[19](#_bookmark43), [21](#_bookmark45)]. Since no compilation error occurs after successfully being applied by Git merge, it would be extremely challenging for developers to notice such a bug, not to mention identify the root cause. With the assistance of WizardMerge, the loss of template specialization can be detected in the merged version, thereby notifying developers to address it manually.

#### 5      Discussion

In this section, we discuss the present limitations in WizardMerge’s design and explore potential avenues for enhancing WizardMerge.

_Improve_ _Graph_ _Generation._ As discussed in §[3.3](#_bookmark6), WizardMerge incorporates three node types and five edge types to construct the ODG. While it adeptly manages numerous merging scenarios, the evaluation outcomes highlighted in

§[4](#_bookmark17) suggest that this incompleteness could result in an inability to address certain conflicts or violated DCB instances. For example, WizardMerge cannot infer the dependency from a global variable to a function if the global variable is initialized by the function. This is because, during the compilation process, the initial values of global variables are established before the execution. In contrast, a function’s return value is accessible only after being executed. When initializing global variables with a function’s return value, the compiler transforms the function invocation into initialization code. To ensure the execution of this initialization code before the main program commences, the compiler positions this code within a specific section of the executable file, commonly denoted as the ".init" segment. This separation facilitates the execution of these initialization routines prior to the initiation of the main program logic. We leave updating the features as our future work to adapt to such scenarios.

_Assess Code Optimization._ WizardMerge conducts dependency analysis based on the compilation metadata. Never-theless, it overlooks a substantial amount of code information due to compilation optimizations like unused function elimination and vectorization [[27](#_bookmark51)]. As an interim measure, WizardMerge presently enforces the "O0" option. However, some projects require compilation optimization for successful building. For instance, the Linux kernel relies on opti-mization to eliminate redundant code segments and disallows the "O0" option. In the future, we aim to explore how to adapt our approach to arbitrary optimization options.

#### 6      Related Work

**6.1**      **Merging Strategies**

_Structured Merging._ Mastery [[45](#_bookmark70)] introduces an innovative structured merging algorithm that employs a dual approach: top-down and bottom-up traversal. It can minimize redundant computations and enable the elegant and

  

efficient handling of the code that has been relocated from its original position to another location. Spork [[23](#_bookmark47)] extends the merging algorithm from the 3DM [[26](#_bookmark50)] to maintain the well-structured code format and language-specific constructs. To fully retain the original functionality of the merged code, SafeMerge [[35](#_bookmark60)] leverages a verification algorithm for analyzing distinct edits independently and combining them to achieve a comprehensive demonstration of conflict freedom.

_Semi-structured Merging._ JDime [[4](#_bookmark28)] introduces a structured merge approach with auto-tuning that dynamically adjusts the merge process by switching between unstructured and structured merging based on the presence of conflicts. FSTMerge [[5](#_bookmark29)] stands as an early example of semi-structured merging, offering a framework for the development of semi-structured merging. Intellimerge [[34](#_bookmark58)] transforms source code into a program element graph via semantic analysis, matches vertices with similar semantics, merges matched vertices’ textual content, and ultimately translates the resulting graph back into source code files.

The above works, however, cannot help the developers with resolving conflicts. On the contrary, WizardMerge is constructed atop Git merge, furnishing developers with clues to aid manual resolution instead of dictating final merging decisions.

#### 6.2      Conflict Resolution

_Conflict_ _Resolution_ _Assistance._ TIPMerge [[9](#_bookmark33)] evaluates the developers’ experience with the key files based on the project and branch history [[9](#_bookmark33)] to determine the most suitable developers for merging tasks. Automerge [[44](#_bookmark69)] relies on version space algebra [[25](#_bookmark49)] to represent the program space and implement a ranking mechanism to identify a resolution within the program space that is highly likely to align with the developer’s expectations. SoManyConflicts [[33](#_bookmark59)] is implemented by modeling conflicts and their interrelations as a graph and employing classical graph algorithms to offer recommendations for resolving multiple conflicts more efficiently.

_Machine Learning_ _Approaches._ MergeBERT [[38](#_bookmark63)] reframes conflict resolution as a classification problem and leverages token-level three-way differencing and a transformer encoder model to build a neural program merge framework. Gmerge [[43](#_bookmark68)] investigates the viability of automated merge conflict resolution through k-shot learning utilizing pre-trained large language models (LLMs), such as GPT-3 [[16](#_bookmark40)]. MergeGen [[13](#_bookmark37)] treats conflict resolution as a generation task, which can produce new codes that do not exist in input, and produce more flexible combinations.

The essential difference compared with existing assistance systems and machine learning approaches is that Wiz-ardMerge considers all modified code snippets, which further helps developers resolve the violated DCBs in merging scenarios.

#### 7      Conclusion

We introduce WizardMerge, a novel code-merging assistance system leveraging the merging results of Git and dependency analysis based on LLVM IR, named definition range, and debug information. Via the dependency analysis, WizardMerge is able to detect the loss of dependency of the named definitions, therefore providing more accurate merging order suggestions including conflicts and applied code blocks for the developers. Our evaluation demonstrates that WizardMerge significantly diminishes conflict merging time costs. Beyond addressing conflicts, WizardMerge provides merging suggestions for most of the code blocks potentially affected by the conflicts. Moreover, Wizard-Merge exhibits the capability to identify conflict-unrelated code blocks which should require manual intervention yet automatically applied by Git.

  

#### References

[1]   Paola Accioly, Paulo Borba, and Guilherme Cavalcanti. 2018. Understanding semi-structured merge conflict characteristics in open-source java projects. _Empirical_ _Software_ _Engineering_ 23 (2018), 2051–2085.

[2]   Iftekhar Ahmed, Caius Brindescu, Umme Ayda Mannan, Carlos Jensen, and Anita Sarma. 2017. An empirical examination of the relationship between code smells and merge conflicts. In _2017 ACM/IEEE International Symposium on Empirical Software Engineering and Measurement (ESEM)_. IEEE, 58–67.

[3]   Waad Aldndni, Na Meng, and Francisco Servant. 2023. Automatic prediction of developers’ resolutions for software merge conflicts. _Journal_ _of_ _Systems and Software_ 206 (2023), 111836.

[4]   Sven Apel, Olaf Leßenich, and Christian Lengauer. 2012. Structured merge with auto-tuning: balancing precision and performance. In _Proceedings_ _of_ _the 27th IEEE/ACM International Conference on Automated Software Engineering_. 120–129.

[5]   Sven Apel, Jörg Liebig, Benjamin Brandl, Christian Lengauer, and Christian Kästner. 2011. Semistructured merge: rethinking merge in revision control systems. In _Proceedings of the 19th ACM SIGSOFT symposium and the 13th European conference on Foundations of software engineering_. 190–200.

[6]   ApsarasX. 2021. Use LLVM by JavaScript/TypeScript. [https://dev.to/apsarasx/use-llvm-by-javascripttypescript-27c3](https://dev.to/apsarasx/use-llvm-by-javascripttypescript-27c3). Accessed: 2024-Feb.

[7]   Atlassian. 2022. Git Merge. [https://www.atlassian.com/git/tutorials/using-branches/git-merge](https://www.atlassian.com/git/tutorials/using-branches/git-merge). Accessed: 2023-Oct.

[8]   Guilherme Cavalcanti, Paulo Borba, and Paola Accioly. 2017. Evaluating and improving semistructured merge. _Proceedings of the ACM on Programming Languages_ 1, OOPSLA (2017), 1–27.

[9]   Catarina Costa, Jair Figueiredo, Leonardo Murta, and Anita Sarma. 2016. TIPMerge: recommending experts for integrating changes across branches. In _Proceedings of the 2016 24th ACM SIGSOFT International Symposium on Foundations of Software Engineering_. 523–534.

[10]  Cp-Algorithms. 2023. Segment Tree. [https://cp-algorithms.com/data_structures/segment_tree.html](https://cp-algorithms.com/data_structures/segment_tree.html). Accessed: 2023-Oct.

[11]  Elizabeth Dinella, Todd Mytkowicz, Alexey Svyatkovskiy, Christian Bird, Mayur Naik, and Shuvendu Lahiri. 2022. Deepmerge: Learning to merge programs. _IEEE_ _Transactions_ _on_ _Software_ _Engineering_ 49, 4 (2022), 1599–1614.

[12]  Jinhao Dong, Qihao Zhu, Zeyu Sun, Yiling Lou, and Dan Hao. [n. d.]. Merge Conflict Resolution: Classification or Generation? ([n. d.]).

[13]  Jinhao Dong, Qihao Zhu, Zeyu Sun, Yiling Lou, and Dan Hao. 2023. Merge Conflict Resolution: Classification or Generation?. In _2023 38th IEEE/ACM_ _International Conference on Automated Software Engineering (ASE)_. IEEE, 1652–1663.

[14]  Changbin Du. 2018. kernel hacking: GCC optimization for better debug experience (-Og). [https://lwn.net/Articles/753639/](https://lwn.net/Articles/753639/). Accessed: 2024-Jan.

[15]  Paulo Elias, Heleno de S Campos Junior, Eduardo Ogasawara, and Leonardo Gresta Paulino Murta. 2023. Towards accurate recommendations of merge conflicts resolution strategies. _Information_ _and_ _Software_ _Technology_ (2023), 107332.

[16]  Luciano Floridi and Massimo Chiriatti. 2020. GPT-3: Its nature, scope, limits, and consequences. _Minds and Machines_ 30 (2020), 681–694.

[17]  Git. 2023. Git. [https://git-scm.com/](https://git-scm.com/). Accessed: 2023-Oct.

[18]  Git. 2023. Git Merge. [https://git-scm.com/docs/git-merge](https://git-scm.com/docs/git-merge). Accessed: 2023-Oct.

[19]  Istvan Haller, Yuseok Jeon, Hui Peng, Mathias Payer, Cristiano Giuffrida, Herbert Bos, and Erik van der Kouwe. 2016. TypeSan: Practical Type Confusion Detection. In _Proceedings_ _of the_ _2016_ _ACM_ _SIGSAC_ _Conference_ _on_ _Computer and_ _Communications_ _Security_ (Vienna, Austria) _(CCS_ _’16)_.

Association for Computing Machinery, New York, NY, USA, 517–528. [https://doi.org/10.1145/2976749.2978405](https://doi.org/10.1145/2976749.2978405)

[20]  Kumaran Handy. 2023. The Biggest Codebases in History, as Measured by Lines of Code. [https://www.artstation.com/blogs/dioeye/Rn8z/the-](https://www.artstation.com/blogs/dioeye/Rn8z/the-biggest-codebases-in-history-as-measured-by-lines-of-code)[biggest-codebases-in-history-as-measured-by-lines-of-code](https://www.artstation.com/blogs/dioeye/Rn8z/the-biggest-codebases-in-history-as-measured-by-lines-of-code). Accessed: 2024-Jan.

[21]  Yuseok Jeon, Priyam Biswas, Scott Carr, Byoungyoung Lee, and Mathias Payer. 2017. HexType: Efficient Detection of Type Confusion Errors for C++. In _Proceedings_ _of_ _the_ _2017_ _ACM_ _SIGSAC_ _Conference_ _on_ _Computer_ _and_ _Communications_ _Security_ (Dallas, Texas, USA) _(CCS_ _’17)_. Association for Computing Machinery, New York, NY, USA, 2373–2387. [https://doi.org/10.1145/3133956.3134062](https://doi.org/10.1145/3133956.3134062)

[22]  Jatin Khurana. 2014. GIT merge successfully but introduce a compilation error. [https://stackoverflow.com/questions/25270430/git-merge-successfully-](https://stackoverflow.com/questions/25270430/git-merge-successfully-but-introduce-a-compilation-error)[but-introduce-a-compilation-error](https://stackoverflow.com/questions/25270430/git-merge-successfully-but-introduce-a-compilation-error). Accessed: 2024-Jan.

[23]  Simon Larsén, Jean-Rémy Falleri, Benoit Baudry, and Martin Monperrus. 2022. Spork: Structured Merge for Java with Formatting Preservation. _IEEE_ _Transactions on Software Engineering_ 49, 1 (2022), 64–83.

[24]  Chris Lattner. 2008. LLVM and Clang: Next generation compiler technology. In _The_ _BSD_ _conference_, Vol. 5. 1–20.

[25]  Tessa Lau, Steven A Wolfman, Pedro Domingos, and Daniel S Weld. 2003. Programming by demonstration using version space algebra. _Machine_ _Learning_ 53 (2003), 111–156.

[26]  Tancred Lindholm. 2004. A three-way merge for XML documents. In _Proceedings_ _of_ _the_ _2004_ _ACM_ _symposium_ _on_ _Document_ _engineering_. 1–10.

[27]  LLVM. 2023. LLVM’s Analysis and Transform Passes. [https://llvm.org/docs/Passes.html](https://llvm.org/docs/Passes.html). Accessed: 2023-Dec.

[28]  LLVM. 2024. The LLVM Compiler Infrastructure. [https://llvm.org/](https://llvm.org/). Accessed: 2024-Apr.

[29]  Mozilla. 2023. Mercurial Gecko Repositories. [https://hg.mozilla.org/](https://hg.mozilla.org/). Accessed: 2023-Dec.

[30]  Mozilla. 2024. Gecko-dev: nsCOMPtr.h. [https://github.com/mozilla/gecko-dev/blob/a20620423a5363cf7afdac81e062bc687c29366a/xpcom/base/](https://github.com/mozilla/gecko-dev/blob/a20620423a5363cf7afdac81e062bc687c29366a/xpcom/base/nsCOMPtr.h) [nsCOMPtr.h](https://github.com/mozilla/gecko-dev/blob/a20620423a5363cf7afdac81e062bc687c29366a/xpcom/base/nsCOMPtr.h) Accessed: 2024-01-29.

[31]  MySQL. 2024. MySQL. [https://dev.mysql.com/](https://dev.mysql.com/). Accessed: 2024-Apr.

[32]  Yuichi Nishimura and Katsuhisa Maruyama. 2016. Supporting merge conflict resolution by using fine-grained code change history. In _2016 IEEE 23rd_ _International Conference on Software Analysis, Evolution, and Reengineering (SANER)_, Vol. 1. IEEE, 661–664.

  

[33]  Bo Shen, Wei Zhang, Ailun Yu, Yifan Shi, Haiyan Zhao, and Zhi Jin. 2021. SoManyConflicts: Resolve many merge conflicts interactively and systematically. In _2021 36th IEEE/ACM International Conference on Automated Software Engineering (ASE)_. IEEE, 1291–1295.

[34]  Bo Shen, Wei Zhang, Haiyan Zhao, Guangtai Liang, Zhi Jin, and Qianxiang Wang. 2019. IntelliMerge: a refactoring-aware software merging technique. _Proceedings of the ACM on Programming Languages_ 3, OOPSLA (2019), 1–28.

[35]  Marcelo Sousa, Isil Dillig, and Shuvendu K Lahiri. 2018. Verified three-way program merge. _Proceedings_ _of_ _the_ _ACM_ _on_ _Programming_ _Languages_ 2, OOPSLA (2018), 1–29.

[36]  Calvin Spealman. 2018. When a Clean Merge is Wrong. [https://www.caktusgroup.com/blog/2018/03/19/when-clean-merge-wrong/](https://www.caktusgroup.com/blog/2018/03/19/when-clean-merge-wrong/). Accessed: 2024-Jan.

[37]  Alexey Stepanov. 2020. What’s the difference between Kotlin/Native and Java bytecode with compile through LLVM frontend? [https://stackoverflow.](https://stackoverflow.com/questions/60580188/whats-the-difference-between-kotlin-native-and-java-bytecode-with-compile-throu) [com/questions/60580188/whats-the-difference-between-kotlin-native-and-java-bytecode-with-compile-throu](https://stackoverflow.com/questions/60580188/whats-the-difference-between-kotlin-native-and-java-bytecode-with-compile-throu). Accessed: 2024-Feb.

[38]  Alexey Svyatkovskiy, Sarah Fakhoury, Negar Ghorbani, Todd Mytkowicz, Elizabeth Dinella, Christian Bird, Jinu Jang, Neel Sundaresan, and Shuvendu K Lahiri. 2022. Program merge conflict resolution via neural transformers. In _Proceedings of the 30th ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering_. 822–833.

[39]  Robert Tarjan. 1972. Depth-first search and linear graph algorithms. _SIAM journal_ _on computing_ 1, 2 (1972), 146–160.

[40]  Linus Torvalds. 2023. Linux Kernel Official Repository. [https://github.com/torvalds/linux](https://github.com/torvalds/linux). Accessed: 2023-Oct.

[41]  David Vandevoorde and Nicolai M Josuttis. 2002. _C++_ _templates: The complete guide, portable documents_. Addison-Wesley Professional.

[42]  Alexis Määttä Vinkler. 2023. The Magic of 3-Way Merge. [https://blog.git-init.com/the-magic-of-3-way-merge/](https://blog.git-init.com/the-magic-of-3-way-merge/). Accessed: 2023-Oct.

[43]  Jialu Zhang, Todd Mytkowicz, Mike Kaufman, Ruzica Piskac, and Shuvendu K Lahiri. 2022. Using pre-trained language models to resolve textual and semantic merge conflicts (experience paper). In _Proceedings of the 31st ACM SIGSOFT International Symposium on Software Testing and Analysis_. 77–88.

[44]  Fengmin Zhu and Fei He. 2018. Conflict resolution for structured merge via version space algebra. _Proceedings of the ACM on Programming Languages_ 2, OOPSLA (2018), 1–25.

[45]  Fengmin Zhu, Xingyu Xie, Dongyu Feng, Na Meng, and Fei He. 2022. Mastery: Shifted-Code-Aware Structured Merging. In _International Symposium_ _on Dependable Software Engineering: Theories, Tools, and Applications_. Springer, 70–87.