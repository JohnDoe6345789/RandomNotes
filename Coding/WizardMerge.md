#### WizardMerge - Save Us From Merging Without Any Clues

##### QINGYU ZHANG,The University of Hong Kong, China

##### JUNZHE LI,The University of Hong Kong, China

##### JIAYI LIN,The University of Hong Kong, China

##### JIE DING,The University of Hong Kong, China

##### LANTENG LIN,Beijing University of Posts and Telecommunications, China

##### CHENXIONG QIAN∗,The University of Hong Kong, China

```
Modern software development necessitates efficient version-oriented collaboration among developers. While Git is the most popular
version control system, it generates unsatisfactory version merging results due to textual-based workflow, leading to potentially
unexpected results in the merged version of the project. Although numerous merging tools have been proposed for improving merge
results, developers remain struggling to resolve the conflicts and fix incorrectly modified code without clues. We presentWizardMerge,
an auxiliary tool that leverages merging results from Git to retrieve code block dependency on text and LLVM-IR level and provide
suggestions for developers to resolve errors introduced by textual merging. Through the evaluation, we subjectedWizardMergeto
testing on 227 conflicts within five large-scale projects. The outcomes demonstrate thatWizardMergediminishes conflict merging
time costs, achieving a 23.85% reduction. Beyond addressing conflicts,WizardMergeprovides merging suggestions for over 70% of
the code blocks potentially affected by the conflicts. Notably,WizardMergeexhibits the capability to identify conflict-unrelated code
blocks that require manual intervention yet are harmfully applied by Git during the merging.
CCS Concepts:•Software and its engineering→Software configuration management and version control systems;Auto-
mated static analysis;•Theory of computation→Program analysis.
Additional Key Words and Phrases: Version control, Code merging, Static analysis
ACM Reference Format:
Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian. 2024. WizardMerge - Save Us From Merging Without
Any Clues. 1, 1 (July 2024), 22 pages. https://doi.org/10.1145/nnnnnnn.nnnnnnn
```
```
1 Introduction
As the most globally renowned version control system, Git [ 17 ] has been helping with the development and maintenance
of millions of projects. To automatically merge the codes from two different versions, Git utilizes a Three-way Merge
Algorithm [ 42 ] as the default strategy on the text level. The textual-based merging is general and suitable for various...





∗Corresponding author.
```
```
Authors’ Contact Information: Qingyu Zhang, The University of Hong Kong, Hong Kong, China, z1anqy@connect.hku.hk; Junzhe Li, The University of
Hong Kong, Hong Kong, China, jzzzli@connect.hku.hk; Jiayi Lin, The University of Hong Kong, Hong Kong, China, linjy01@connect.hku.hk; Jie Ding,
The University of Hong Kong, Hong Kong, China, dingjie999888@icloud.com; Lanteng Lin, Beijing University of Posts and Telecommunications, Beijing,
China, lantern@bupt.edu.cn; Chenxiong Qian, The University of Hong Kong, Hong Kong, China, cqian@cs.hku.hk.
Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not
made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components
of this work owned by others than the author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on
servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from permissions@acm.org.
©2024 Copyright held by the owner/author(s). Publication rights licensed to ACM.
Manuscript submitted to ACM
```
## arXiv:2407.02818v1 [cs.SE] 3 Jul 2024


2 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

types of text files and the merging tempo is proved to be the fastest. However, it simply considers lines of code, neglecting
all syntax and semantic information, which leads to potential bugs in the merged version of the project [22, 36].
To mitigate the shortcomings of traditional textual-based merging, multiple merging approaches [ 4 , 5 , 23 , 34 , 35 , 44 , 45 ]
are proposed for enhancing the correctness of merging results. These tools can be classified asStructured Merging
andSemi-structured Merging. Structured merging tools [ 23 , 35 , 44 , 45 ] transform the code intended for merging
into an Abstract Syntax Tree (AST), thereby converting the code merging into the merging of edges and nodes on the
AST [ 1 ]. Semi-structured merging [ 4 , 5 , 34 ], akin to structured merging, designates certain semantically insensitive
AST nodes (e.g., strings) as text, allowing them to be directly processed by the text-based merging strategy during the
merging process [8].
Despite structured and semi-structured merging tools facilitating code merging, challenges arise when it is unclear
which version of the code should take precedence. This uncertainty leads to merging conflicts. In the context of large-
scale software projects, conflicts can become a source of frustration for developers, as they must dedicate significant
effort to resolve the issues that arise from merging before they can release the final merged version [ 2 , 32 ]. Addressing
these conflict resolutions has given rise to merging assistance systems. These systems are designed with objectives such
as selecting the most suitable developers to resolve conflicts [ 9 ], prioritizing the resolution order for conflicts [ 33 ], and
even automatically generating alternative conflict resolutions [ 44 ]. With the rapid advancements in machine learning
technologies in recent years, researchers have turned to machine learning approaches [ 3 , 11 – 13 , 15 , 38 , 43 ]. These
systems are trained using extensive datasets containing merge conflict scenarios and can provide predictions on the
most suitable resolution strategy for each conflicting portion of code.
While these researches have made noteworthy contributions to code merging, they still exhibit the following
limitations. Firstly, structured [ 4 , 23 , 35 , 44 , 45 ] and semi-structured [ 5 , 34 ] still generate conflicts when the direct
merging between two AST noes failed, and cannot assist developers in resolving them. Secondly, conflict resolution
assistance systems [ 9 , 33 , 44 ] focus solely on resolving conflicts, overlooking potential errors introduced by incorrectly
applied but conflict-free code. Though machine learning approaches [ 3 , 11 , 12 , 15 , 38 , 43 ] can automate conflict merging,
the functionality of the resulting code may not ensure alignment with developers’ expectations. Additionally, machine
learning methods necessitate specific code datasets for pre-training, which raises universality issues when applied to
more diverse code projects. Moreover, the developers often have unpredictable operations on resolving conflicts, where
the auto-generated conflict resolution may fail to meet the expectations of them.
To address these limitations, we introduce a novel approach aiming to aid developers in handling complex code-
merging scenarios. Recognizing thatdevelopers have unpredictable ultimate needs, our approach focuses on
grouping conflicts by relevance and providing prioritized suggestions for resolving conflicts rather than resolving them
directly. Our approach also detects the incorrectly applied codes as they can be easily neglected but cumbersome to
handle for developers. To realize these design goals, our approach requires a thorough understanding of the dependency
relations in the two merging candidate versions. Firstly, we perform definition dependency graph construction on
LLVM Intermediate Representation (IR) [ 28 ] and utilize definition indicators extracted from Clang [ 24 ] to indicate the
definition name and ranges in the source file. Following graph construction, our approach analyzes the Git merge
outcomes that present all the conflicts and modified code blocks. Then, we align the definitions in the source file with
code snippets in Git’s outcomes to map from LLVM-IR to the Git merge results. Given the dependency graph analysis
and aligned Git merge results, we further conduct an edge-sensitive algorithm to detect incorrectly applied codes, which
we deem as conflicts since they require developers’ attention as well. Finally, we assign priorities to these conflicted
nodes and produce merging suggestions.


WizardMerge - Save Us From Merging Without Any Clues 3

We implement a prototypeWizardMergefor C/C++ projects as most large-scale projects are written in C/C++ [ 20 ].
We evaluateWizardMergeon 227 conflicts from five popular large projects. The outcomes demonstrate thatWiz-
ardMergediminishes conflict merging time costs. Beyond addressing conflicts,WizardMergeprovides merging
suggestions for most of the code blocks affected by the conflicts. Moreover,WizardMergeexhibits the capability to
identify conflict-unrelated code blocks which should require manual intervention yet automatically applied by Git.
In summary, we make the following contributions:

- We proposed a novel approach to infer the dependency between text-level code blocks with modification.
- We proposed a remarkable methodology, which aims to pinpoint code blocks that have been automatically applied
    by Git but introduce unexpected results.
- We implemented a code merging auxiliary prototype namedWizardMergebased on the approaches and evaluated
    the system on projects with large-scale code bases. We will open sourceWizardMergeand evaluation datasets at
    https://github.com/aa/bb.

2 Background

2.1 Git Merging

##### 𝐶 1 𝐶 2

##### 𝐶 4

```
V1 = Stmt1();
V2 = Stmt2(V1);
V3 = Stmt3(V1, V2);
V4 = Stmt2(V2);
V5 = Stmt4(V2, V3);
V6 = Stmt5(V1);
V7 = Stmt6(V6);
```
##### Base

```
V2 = Stmt1();
V1 = Stmt2(V2);
V4 = Stmt2(V2);
V3 = Stmt3(V1, V2);
V5 = Stmt4(V2, V3);
V6 = Stmt5(V1);
V7 = Stmt7(V5, V6);
```
#### Variant A （ VA ）

```
V1 = Stmt1();
V2 = Stmt2(V1);
V3 = Stmt3(V1, V2);
V5 = Stmt4(V2, V3);
V6 = Stmt5(V1);
V7 = Stmt2(V5);
```
#### Variant B （ VB ）

```
V2 = Stmt1();
V1 = Stmt2(V2);
V4 = Stmt2(V2);
V3 = Stmt3(V1, V2);
V5 = Stmt4(V2, V3);
V6 = Stmt5(V1);
<Conflict>
```
##### Merged Commit

```
Line 1
Line 2
```
```
×
```
##### 𝐶 3 𝑽𝒂𝒓𝒊𝒂𝒏𝒕^ 𝑨

##### 𝐶 5 𝑽𝒂𝒓𝒊𝒂𝒏𝒕^ 𝑩

```
𝑩𝒂𝒔𝒆
```
##### 𝐶 6

```
𝑴𝒆𝒓𝒈𝒆𝒅
𝑪𝒐𝒎𝒎𝒊𝒕
Developer
```
```
Line 3
Line 4
Line 5
Line 6
Line 7
```
```
DCB 1 DCB 2
```
```
DCB 3
```
```
DCB 4
×
```
```
Fig. 1. Three-way merging workflow of Git.
```
Git is a free and open-source distributed version control system designed for efficient handling of projects of all
sizes [ 17 ]. Git merge, as a key operation, facilitates the integration of changes between versions and is often used to
consolidate code changes, bug fixes, or new features [ 18 ]. Git supports multiple textual-based merging algorithms [ 7 ].
Take the default and most commonly employed algorithm, Three-way Merging Algorithm [ 42 ] as an example, the
workflow is shown in Figure 1. When the developer tries to merge two commits (i.e.,𝐶 3 Variant A (VA)and𝐶 5 Vairant
B (VB)in Figure 1), Git first traverses the commit-graph back until it reaches the nearest common ancestor commit (i.e.,
𝐶 2 , namedBase). Then, all source code blocks of each newer version are mapped to the corresponding code blocks in
base versions, and the modified code blocks of either VA or VB are regarded asDiff Code Blocks (DCBs). A DCB in a


4 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

variant may contain zero or multiple lines of code, indicating the removal of an existing line or modification/addition of
multiple continuous lines. After the code matching stage, Git will iterate each code block tuple, and determine which
commit should be applied according to the following rules:

- If code blocks ofVAandVBremain the same as the code block ofBase, or both of them are changed in the same
    way, inherit the code block fromVA. In Figure 1, both VA and VB remove the definition ofv4(i.e., deleting line 4 of
       Base). Due to identical modifications at the same locations, Git applies these changes. It is noteworthy that even for
       code removal, VA and VB maintain a pair of empty DCBs, symbolizing the deletion of lines.
- If only one code block of eitherVAorVBis changed, Git applies this code block, and marks this code block and the
    corresponding code block in the other variant as DCBs. In Figure 1, VA alters lines 1-2 and introduces a new line at
    3. Git’s code-matching algorithm identifies these modifications, designating lines 1-3 of VA asDCB 1and lines 1-2 of
    VB asDCB 2. Since onlyDCB 1changes, Git applies it to generate the code for the Merged Commit at lines 1-3.
- If both code blocks fromVAandVBare changed, and the modifications are different, Git marks both code blocks as
    DCBs. As Git cannot decide which one should be applied, it emits a conflict containing the two DCBs. In Figure 1, VA
    and VB independently modify the definition ofv7. Git detects these alterations, marking line 7 of VA asDCB 3and
    line 6 of VB asDCB 4. Since these two modifications differ, Git cannot determine which one to apply. Consequently,
    Git signals the inconsistency betweenDCB 3andDCB 4, prompting developers to resolve this conflict.

### int a = 1;

### int b = 2;

### int c = 1 + b;

### printf(“%d”, c);

### // int a = 1;

### int b = 2;

### int c = 1 + b;

### printf(“%d”, c);

### int a = 1;

### int b = 2;

### int c = a + b;

### printf(“%d”, c);

### VA Base

### VB

### // int a = 1;

### int b = 2;

### int c = a + b;

### printf(“%d”, c);

### Merged

```
Fig. 2. An example of a clean but incorrect Git merge with the three-way merging algorithm
```
2.2 Limitations of Existing Approaches

Despite being the most widely used code merging tool currently, Git merge faces the following two challenges:(Challenge
1) It cannot ascertain developers’ expectations for code functionality.Consequently, when both versions modify the same
code, Git treats this discrepancy as a conflict, necessitating manual intervention for resolution. We discussed how the
conflicts are generated in §2.1.(Challenge 2) It cannot ensure the correctness of successfully applied and conflict-free code
blocks.Even when Git successfully merges two versions without conflicts, the merged code may be generated with
new bugs. Figure 2 shows an incorrect merging scenario. Only the first line ofVAand the third line ofVBare changed.


WizardMerge - Save Us From Merging Without Any Clues 5

Using Git merge, the two versions of code can be easily merged without any conflict (i.e., the first line applied from
VA and the second line applied from VB). Unfortunately, in the merged version, the definition of variableahas been
removed, which causes a compilation error at the third line. These DCBs enforced by Git without conflict, yet resulting
in unexpected results, are termedViolated DCBs. Given that these violated DCBs are not flagged as conflicts and
are automatically incorporated by Git merge, they tend to introduce concealed programming errors into the merged
code. In the most severe scenarios, these error-prone code segments might seemingly execute successfully but could
potentially give rise to security issues post-deployment [22, 36].
Over the years, researchers have concentrated on enhancing the quality of merged code and conflict resolution.
However, these two basic limitations have not yet been fully resolved. On the one hand, structured [ 23 , 35 , 44 , 45 ] and
semi-structured [ 4 , 5 , 34 ] merging still report conflicts when the same AST node is modified simultaneously by two
merging candidate branches. On the other hand, the conflict resolution assistance system [ 33 , 44 ] and machine learning
approaches [ 3 , 11 – 13 , 15 , 38 , 43 ] only consider conflicts reported by the merge tool, ignoring all code snippets that
have been applied. Additionally, automatic software merging tools [ 3 – 5 , 11 – 13 , 15 , 23 , 34 , 35 , 38 , 43 – 45 ] neglect an
essential observation:developers have unpredictable ultimate needs, therefore cannot fully overcomeChallenge 1.
To better illustrate this, we provide an example of implementing the Fibonacci sequence in Figure 3. Two versions, A
and B, calculate the Fibonacci sequence using recursion and iteration, respectively. When attempting to merge these
versions, a conflict arises, and two developers resolve it. Although both developers decide to use the iterative approach
from version B to avoid stack overflow for large values of𝑛, they implement different strategies for handling negative
values of𝑛. Developer A, inspired by version A, uses an unsigned definition for𝑛to eliminate negative values (line 1
in resolution A). In contrast, Developer B opts to return zero when𝑛is less than zero (line 3 in resolution B). Since
developers’ decisions on conflict resolution are unpredictable, automatic software merging tools often fail to produce
an acceptable resolution in such cases.

```
Fig. 3. An example of different resolutions to the same conflict
```
Furthermore, machine learning systems [ 3 , 11 – 13 , 15 , 38 , 43 ] are limited in their ability to resolve conflicts in large
codebases due to input and output sequence constraints. However, these conflicts typically do not significantly impact


6 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

developers’ time. For example, even MergeGen [ 13 ], the most advanced system available, failed to achieve satisfactory
results when applied to the large-scale targets evaluated in §4. Upon completing MergeGen’s pre-training, we discovered
that it produced no correct outputs, achieving an accuracy of 0%. This failure can be attributed to MergeGen’s default
input and output sequence length limitations of 500 and 200 BPE, respectively [ 13 ]. These limitations are excessively
stringent for large-scale project source code, leading to truncation of code beyond the prescribed lengths. However, the
end of the truncated code is usually far away from where conflict occurs and MergeGen cannot master the ability to
resolve the conflicts given the trimmed training and testing sets. Attempts to relax these length limitations are impractical
due to increased GPU memory consumption during training. Moreover, while machine learning techniques excel in
generating merge results, they may not always align perfectly with developers’ expectations. Although MergeGen’s
evaluation results demonstrate superior conflict code matching precision and accuracy on a line of code basis compared
to previous works [ 13 ], it still falls short of expectations in some cases, with average precision and accuracy rates of
71.83% and 69.93%, respectively.

3 Design

Given the challenges outlined in the preceding section, we have identified two key insights. AddressingChallenge
1 , we recognize that the expected outcomes of merges are subjective and open-ended. Consequently, our approach
emphasizes guiding developers in resolving conflicts rather than prescribing resolutions. To tackleChallenge 2, it is
imperative to analyze all modified code. To this end, we presentWizardMerge, a novel conflict resolution assistance
system, which provides developers with clues and aids in conflict resolution. Moreover,WizardMergeemphasizes all
modified code snippets, not just conflicts, enabling it to uncover hidden discrepancies and preempt potential errors. A
detailed design ofWizardMergewill be presented in the subsequent subsections.

3.1 Overview

```
Source Code (VA)
```
```
Source Code (VB)
```
###### Target Project

```
LLVM IR Def Indicator
```
###### VA’s Metadata

```
LLVM IR Def Indicator
```
###### VB’s Metadata

```
Compilation
```
```
Merge Result
Git Merge
```
###### Dependency Graph Generation

```
Node
Generation
```
```
Dependency
Analysis
```
###### Resolving Suggestion Generation

```
Graph-Text
Alignment
Violated DCB
Detection
```
```
Priority-oriented
Classification
```
```
Overall Dependency Graph
```
###### Group 1

```
DCB a
DCB b
...
```
# ...

###### Group 2

```
DCB h
DCB j
...
```
###### Group N

```
DCB x
DCB y
...
```
Manuscript submitted to ACM Fig. 4. System overview ofWizardMerge.


WizardMerge - Save Us From Merging Without Any Clues 7

Figure 4 depicts the system overview ofWizardMerge.WizardMergefirst consumes compilation metadata from
the two merge candidates through LLVM[ 28 ].WizardMergeemploys LLVM based on two insights: i) For large
projects, code is compiled before a new version release to ensure compilation success, allowingWizardMergeto
collect metadata without introducing additional merging steps; ii) LLVM Intermediate Representation (IR) provides
a robust static analysis framework and comprehensive debug information, facilitatingWizardMergein performing
definition dependency reasoning. To bridge LLVM IR analysis with merge results from Git, the compilation metadata
also encompasses definition indicators. A definition indicator serves as the representation of a named definition section
at the source code level, indicating the range and name of a definition. For example, a definition indicator of a function
records its function name, along with its start and end line numbers in the source code file. Subsequently, To identify
relationships between all modified codes,WizardMergeadvances to theDependency Graph Generationstage.
Instead of focusing only on conflicts, dependencies for all definitions are analyzed.WizardMergecreates vertices for
the definitions and establishes edges between vertices according to the dependency relationships of definitions analyzed
from LLVM IRs. Following this, theOverall Dependency Graph (ODG)is generated to represent the dependencies
among all the definitions in both candidate versions of the target project. To attach Git’s textual results with IR-level
dependency information,WizardMergealigns Git merge’s results and ODG to match DCBs with vertices in ODG
throughGraph-Text Alignment. This stage is capable of mapping DCBs to the corresponding definition nodes in each
ODG, refining definition nodes into DCB nodes, and transforming definition dependencies into DCB dependencies.
To rectify errors introduced by Git merge,WizardMergeemploys an edge-sensitive algorithm in theViolated DCB
Detectionstage, detecting both conflict-related and conflict-unrelated violated DCBs. These violated DCBs and conflicts
are categorized, prioritized based on the dependency graph, and presented as suggestions to developers to assist in
resolution.

3.2 Dependency Graph Generation

WizardMergefirst constructs ODG based on the LLVM IR and definition indicator.WizardMergecurrently supports
the following types of nodes and edges:
Graph node.Subsequently,WizardMergegenerates nodes for each named definition with the corresponding indicator.
Considering the types of definitions,WizardMergecreates three types of nodes based on the indicator information:
Type Node (TN)represents composite types, e.g. structure, class, type alias, and enumeration;Global Variable Node
(GN)represents global variable;Function Node (FN)represents function.
Graph edge.Each named definition may depend on other definitions within the project. For instance, a function
might utilize a global variable or return a pointer with a specific structure type. In the ODG, these usage dependencies
among definitions are deduced from LLVM IRs and represented by the edges linking one node to another. The types of
edges are defined by the types of their two connected nodes. An edge of type A-B represents a dependency from node
A to node B. InWizardMerge, five classifications of edges are supported: TN-TN, GN-TN, FN-TN, FN-GN and FN-FN.
Note thatWizardMergewill never generate an edge from TN to FN, even though member functions may be defined
within a composite type. Instead,WizardMergecreates TN for all functions, irrespective of whether the function is
standalone or nested within another scope. If an FN represents a nested function defined inside a composite type, an
FN-TN edge is built to represent the dependency from the function to the specific type.

3.3 Resolving Suggestion Generation

In this stage,WizardMergeextracts DCB information from Git merge to aid in the analysis. Then, merging suggestions
can be generated based on the analysis results.


8 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

3.3.1 Graph-Text Alignment.The merge results cannot be directly linked to the corresponding node in ODG. This is
primarily because these results are raw texts that provide no additional clues for program analysis. In the raw results, a
single large DCB may cover multiple nodes and a single node may encompass several DCBs. This N-to-N relationship
complicates the analysis of the graph.

```
Name: Named Def 2
Start Line: L 6
End Line: L
DCB: [L 6 , L 7 ]
```
```
L
L
L
L
L
L
L
```
```
Named Def 1 {
+ ...
+ };
+
+ Named Def 2 {
+ ...
+ };
```
###### Code File with DCBs

###### Before Graph-Text Alignment After Graph-Text Alignment

```
Name: Named Def 1
Start Line: L
End Line: L
```
```
Name: Named Def 2
Start Line: L
End Line: L
```
```
Name: Named Def 1
Start Line: L
End Line: L
DCB: [L2, L5]
```
```
Name: Named Def 2
Start Line: L
End Line: L
DCB: [L2, L5]
```
```
Fig. 5. Example alignment between node and DCB text.
```
WizardMergeinitially associates each DCB with its corresponding definition indicators. To identify all nodes
contained in the file where the DCB is situated,WizardMergeextracts the nodes whose range intersects with the
DCB range. If no matched node is found, it indicates that the DCB is not in a definition (e.g., newlines or comments)
and will not be further analyzed byWizardMerge. Treating each DCB as a "line segment", its start and end line
numbers represent the coordinates of the line segment’s two endpoints on the coordinate axis. Similarly, each definition
indicator of a node is considered as a unique "color". The range of each node signifies a specific coordinate interval that
is colored with the definition represented by the node. Thus, retrieving the node during each DCB traversal can be seen
as a color query for the interval[𝑠𝑡𝑎𝑟𝑡_𝑙𝑖𝑛𝑒,𝑒𝑛𝑑_𝑙𝑖𝑛𝑒], and interval modification and information maintenance can be
efficiently managed using line segment trees. For a target project containing𝑁lines of code, each "coloring" and "color
query" operation can be achieved with a time complexity of𝑂(𝑙𝑜𝑔 2 𝑁)[ 10 ].WizardMergesubsequently logs all DCBs
corresponding to each node, organizing them in ascending order of line numbers. The node is then subdivided into
finer-grained nodes. The ranges of these finer-grained nodes are updated concurrently based on the original node and
their corresponding unique DCBs. Fine-grained nodes belonging to the same definition will be connected sequentially
in range order, which implicitly expresses the dependency between code blocks. This mechanism ensures that each
node in ODG aligns with at most one DCB, effectively preventing any matching confusion within the ODG.
Taking Figure 5 as an instance, there are two definitions:Named Def 1,Named Def 2. Before alignment, the nodes
are created with their range information. Let’s consider two DCBs associated with these definitions ([𝐿 2 ,𝐿 5 ]and
[𝐿 6 ,𝐿 7 ]). To establish a one-to-one mapping between DCBs and nodes, each node can be divided into multiple ones,
while unaffected lines are excluded as they do not influence the merging. For DCB[𝐿 2 ,𝐿 5 ],WizardMergedetects that
[𝐿 2 ,𝐿 3 ]belongs toNamed Def 1and[𝐿 5 ,𝐿 5 ]belongs toNamed Def 2. Consequently, a new node starting from𝐿 2 to𝐿 3
will replace the original node forNamed Def 1. As[𝐿 1 ,𝐿 1 ]remains unchanged, it will be discarded. Furthermore, a new
node from𝐿 5 to𝐿 5 is also formed since[𝐿 6 ,𝐿 7 ], representing another portion ofNamed Def 2, corresponds to the DCB
[𝐿 6 ,𝐿 7 ]. To indicate the code dependency from lines to previous lines within the same definition, an additional edge
from node[𝐿 6 ,𝐿 7 ]to node[𝐿 5 ,𝐿 5 ]is established.
AsWizardMergeis tasked with managing two code versions (VA and VB), some nodes in the ODG of VA must
possess a corresponding mirror node in the ODG of VB.WizardMergealso constructs a mapping of each node to its


WizardMerge - Save Us From Merging Without Any Clues 9

matched node in the other version’s ODG by matching the name of the definition and the DCB metadata linked to the
node. Note that if one node is aligned with a DCB, it must have one unique mirror node with the other DCB in the
same DCB pair. If the node has a matched node, the matched node must be the mirror node and the definition name is
consistent with the current node. The matched node results contribute to the violated DCB detection via dependency
missing determination. Besides, the ODG’s size can also be shrunk if there are nodes not attached to any DCBs, which
means they are not changed during merging. After the node deletion, the newly generated graph is calledShrunk
Dependency Graph (SDG).

3.3.2 Violated DCB Detection.Violated DCBs are the underlying cause of seemingly successful but erroneous merging.
To offer developers a comprehensive understanding of the merging process,WizardMergedetects violated DCBs
based on an edge-sensitive algorithm and deems them as conflicts requiring resolution suggestions. Given the SDGs
and node matching relationship,WizardMergetraverses each edge in the two versions of the SDG separately. Each
edge, starting from𝑣and ending at𝑢, signifies that𝑣relies on𝑢.WizardMergedetermines which version of the DCB
associated with𝑢and𝑣is ultimately applied in the merge results (i.e., from VA, VB, or conflict).WizardMergethen
ascertains whether these nodes represent violated DCBs by the applied versions and matching nodes of𝑢and𝑣. We
introduce all the safe (i.e. not violated DCBs detected) and violated scenarios as follows.

Symbols Definition.Let the SDGs from VA and VB be represented as𝐺𝐴 =<𝑉(𝐺𝐴),𝐸(𝐺𝐴) >and𝐺𝐵 =<
𝑉(𝐺𝐵),𝐸(𝐺𝐵)>, where𝑉indicates the vertices set and𝐸indicates the edges set. For each vertex𝑣∈𝑉, we represent
its mirror node as𝑀𝑖(𝑣)and use𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑣))to indicate whether the mirror node of𝑣is a matching node (𝑇𝑟𝑢𝑒or
𝐹𝑎𝑙𝑠𝑒).𝑣belongs to only one of the𝑉𝑁,𝑉𝐴, and𝑉𝐶vertex sets, representing not applied, applied, and conflict node sets
respectively. For each edge𝑒(𝑣,𝑢) ∈𝐸, it represents an edge from𝑣to𝑢, indicating𝑣’s dependency on𝑢. According to
the construction principles of SDG, there are two basic facts:

- If𝑣∈𝑉𝐴, then𝑀𝑖(𝑣) ∈𝑉𝑁, and vice versa.
- If𝑣∈𝑉𝐶, then𝑀𝑖(𝑣) ∈𝑉𝐶.
As each𝑣∈𝑉𝐴must indicate another vertex (i.e.,𝑀𝑖(𝑣)) belongs to𝑉𝑁,WizardMergeonly analyzes the cases where
𝑣∈𝑉𝐴or𝑣∈𝑉𝐶to avoid duplicate traversal.

Safe Scenarios.For each𝑣and an edge𝑒=(𝑣,𝑢), we define the scenarios satisfying any of the following four formulas
as safe:
𝑣∈𝑉𝐴,𝑢∈𝑉𝐴 (1)
𝑣∈𝑉𝐴,𝑢∉𝑉𝐴,𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑢))=𝑇𝑟𝑢𝑒 (2)
𝑣∈𝑉𝐶,𝑢∈𝑉𝐴 (3)
𝑣∈𝑉𝐶,𝑢∉𝑉𝐴,𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑢))=𝑇𝑟𝑢𝑒 (4)

In SDGs, the above formulas can be regarded as an edge-sensitive detection algorithm, which is visiualized as depicted
in Figure 6. For Equation 1, if both𝑢and𝑣are applied, it means the dependency relation between nodes within the
same SDG (as shown inScenario 1of Figure 6). By the same token,Scenario 3also illustrates such a direct dependency
represented by Equation 3. For Equation 2 (Scenario 2in Figure 6),𝑣is applied but𝑢is either not applied (𝑢∈𝑉𝑁) or in
conflict status (𝑢∈𝑉𝐶). Nevertheless, if the mirror of𝑢matches𝑢(𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑢))=𝑇𝑟𝑢𝑒), it means that at least in the
other SDG,𝑣can find its corresponding dependency applied (i.e.,𝑀𝑖(𝑢)) although𝑢is not applied, or both versions of
conflicted𝑢hold the same dependency for𝑣. To this end,WizardMergeuses a virtual dependency edge from𝑣to


10 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

𝑀𝑖(𝑢)to indicate a cross dependency between nodes from different SDGs. Similarly, Equation 4 (Scenario 4) is also
regarded as safe with the same reason.

```
Fig. 6. Edge traversal scenarios visualization
```
Violated Scenarios.For each𝑣and an edge𝑒=(𝑣,𝑢), all the scenarios not detected as safe are regarded as violated
scenarios. These scenarios are:
𝑣∈𝑉𝐴,𝑢∉𝑉𝐴,𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑢))=𝐹𝑎𝑙𝑠𝑒 (5)
𝑣∈𝑉𝐶,𝑢∉𝑉𝐴,𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑢))=𝐹𝑎𝑙𝑠𝑒 (6)

TheScenario 5andScenario 6in Figure 6 visualize Equation 5 and Equation 6 respectively. Once𝑢is not applied or in
conflict, and the mirror node of𝑢does not match it (𝑚𝑎𝑡𝑐ℎ(𝑀𝑖(𝑢))=𝐹𝑎𝑙𝑠𝑒), it means that𝑣misses dependency from𝑢
in both SDGs (when𝑢∈𝑉𝑁), or𝑀𝑖(𝑢)holds a different dependency from𝑢, which does not match𝑣’s requirement
(when𝑢∈𝑉𝐶).
WizardMergeidentifies these cases are incorrectly handled by Git merge and marks the corresponding𝑣,𝑢,𝑀𝑖(𝑣)
and𝑀𝑖(𝑢)as violated. Note that vertices with conflicts are always treated as violated, regardless of whether they are
encountered in safe or violated scenarios. Finally, the DCBs attached to these vertices are also marked as violated.


WizardMerge - Save Us From Merging Without Any Clues 11

Fig. 7. MDG example. Each node in MDG contains exactly a pair of mirror nodes in SDG. Each edge is built if there is at least one
SDG edge from one SDG node to another.

3.3.3 Priority-Oriented Classification.WizardMergefurther minimizes the number of nodes that require processing
and constructs aMinimum Dependency Graph (MDG)to emulate the dependencies of the merged version of code.
In the MDG, each node will consist of either an applied node or a conflict node and its matching node, and the edges
between nodes will be created based on various SDGs according to the applying status. Specifically, while establishing
the MDG,WizardMergewill traverse each conflict node or node associated with violated DCBs, and based on the
applying status of the node, it will switch to the corresponding version of SDG and continue traversing the nodes
in that particular SDG. Particularly, if the node is in conflict status, then the outcoming edges of both the mirror of
the node and itself would be iterated. Concurrently,WizardMergesets up a directed edge between corresponding
nodes for the MDG, relying on the encountered edges during traversal. Ultimately, the MDG encapsulates the essential
dependencies of the two versions of code to be merged.
Figure 7 shows an instance illustrating how MDG is built from SDG. Each MDG node is created from a pair of nodes
related to conflict or violated DCBs. SDG nodeA-Eare from VA and allMirrornodes are from VB. Starting from A and
its mirror,WizardMergeiterates the outcoming edges and finds B is linked to a violated DCB. Then,WizardMerge
creates a new nodeMDG_Node_Bcontaining B and its mirror. As B is not applied while its mirror node is applied,
WizardMergeiterates the outcoming edges of the mirror of B and finds a pair of conflict nodes (i.e., E and Mirror),
which formsMDG_Node_E.WizardMergethen iterates the outcoming edges of both B and Mirror because of the
conflict. By the same token,MDG_Node_DandMDG_Node_Care created.
WizardMergethen employs the Tarjan algorithm [ 39 ] to condense the MDG nodes and eliminate cycles. During
this process, nodes in the same strongly connected component are regarded as having identical priority, as these nodes
may reference each other within the code. In the case of the directed acyclic MDG,WizardMergedivides the MDG
into multiple subgraphs based on connectivity. All nodes within each subgraph are grouped, and each node is assigned
a priority through the topological sorting of the subgraph. Nodes that appear later in the topological order signify that
other nodes have greater dependencies on them, while the nodes themselves have fewer dependencies. Consequently,
these nodes are assigned higher priority, enabling developers to address them earlier. In Figure 7,MDG_Node_C,


12 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

MDG_Node_DandMDG_Node_Eare in the same cyclic of MDG and are thus assigned the same resolution priority as
they depend on each other. After calculating the topological order,WizardMergesuggests that the DCB resolution
priority should be {MDG_Node_C, MDG_Node_D, MDG_Node_E}, {MDG_Node_B} and {MDG_Node_A}.

4 Evaluation

In this section, we evaluate the effectiveness ofWizardMerge, aiming to answer the following research questions:

- RQ1:CanWizardMergehelp reduce the time consumption of merging conflicts?
- RQ2:CanWizardMergedetect violated DCBs related to conflicts applied by Git?
- RQ3:CanWizardMergedetect violated DCBs unrelated to conflicts applied by Git?
    Dataset Collection.We choose Firefox [ 29 ], Linux Kernel [ 40 ], MySQL [ 31 ], PHP [ 31 ], and LLVM [ 28 ] as the target
projects because of their widespread usage as open-source software. Additionally, they boast extensive code bases and
are maintained by multiple developers, resulting in repositories that feature intricate commits and merging scenarios.
We collect 22 conflict pairs from Firefox, 43 from Linux Kernel, 17 from LLVM, 46 from PHP, and 99 from MySQL with
a committing timeframe from 2021 to 2024. For each pair of conflicts, we only focus on the files in C/C++ format and
guarantee they either have at least five conflict regions or violated DCBs detected. On average, a conflict contains
824.33 lines of codes and is related to 6.8 files.
Evaluate with State-of-the-art (SOTA) Systems.We initially intended to assessWizardMergealongside other cutting-
edge systems. However, JDIME [ 4 ], Mastery [ 45 ], Spork [ 23 ], SafeMerge [ 35 ], AutoMerge [ 44 ], IntelliMerge [ 34 ] and
SoManyConflicts [ 33 ] encountered language incompatibility issues within our datasets. Besides, migratingWizard-
Mergeto the languages these systems support is impractical because as an LLVM-based project,WizardMergerequires
the full support from the LLVM framework, while the compiler front ends of Java [ 37 ], Typescript [ 6 ], and Javascript [ 6 ]
are still under development. Furthermore, FSTMerge[ 5 ] purports scalability contingent upon users implementing the
merging engine for specific languages. Unfortunately, its C/C++ parsers are either underdeveloped (i.e., accommodating
only basic syntax) or unimplemented, rendering it unsuitable for real-world projects. In addition, as mentioned in § 2.2,
the machine learning approaches [ 3 , 11 – 13 , 15 , 38 , 43 ] meet the sequence length limitation challenge when facing the
large-scale targets we chose.
Evaluation Environment.We evaluatedWizardMergeon an AMD EPYC 7713P server with 64/128 cores/threads,
NVIDIA GeForce RTX 4090 (24GB GPU memory), and 256GB memory, running Ubuntu 20.04 with kernel 5.4.0.

4.1 Effectiveness on Conflict Resolution

To addressRQ1, we conducted a comparative analysis of the time consumption associated with two distinct approaches
(Git merge w/ and w/oWizardMerge) for merging conflict pairs. AsWizardMergeonly provides suggestions instead
of resolutions of conflicts, We have to assign two types of technical staff to the manual merging phase:skilled staffs,
researchers with deep understanding of the project, andordinary staffs,researchers having general understanding of
the project. Initially, the ordinary staff employedWizardMergeto analyze the two versions and performed the merge
utilizing the suggestions provided byWizardMerge. Subsequently, the skilled staff manually merged the two candidate
versions using Git merge without any additional assistance. We meticulously recorded the number of conflicts that can
be analyzed byWizardMergeand the time taken (with precision to the minute) from the commencement of conflict
analysis to their complete resolution.


WizardMerge - Save Us From Merging Without Any Clues 13

Table 1. Conflict coverage ofWizardMerge.Total Numindicates the total number of conflicts in each dataset.Analyzed Num
indicates the number of conflicts successfully analyzed byWizardMerge.Cover Rateindicates the ratio of the above two columns.

```
Dataset Analyzed Num Total Num Cover Rate
Firefox 159 160 99.38%
Linux Kernel 75 88 85.23%
LLVM 71 72 96.61%
PHP 236 255 92.55%
MySQL 797 964 82.68%
```
Before comparing the effectiveness of merging w/ and w/oWizardMerge, we first evaluate its ability to assess
conflicts. As shown in Table 1, we calculated the total number of conflicts for each dataset and recorded how many
conflicts can be successfully analyzed byWizardMerge.WizardMergecan give suggestions of more than 80% conflicts
from the five targets. Notice that some conflicts cannot be handled byWizardMerge, we manually dig the reasons
behind the failure. In the design section, we mentioned thatWizardMerge’s analysis is based on LLVM IR. However,
in textual merging results, some conflicts are caused by the content of comments or preprocessing instructions (e.g.,
#includeand macro definition). Since comments do not belong to the code scope, and the preprocessing instructions will
be expanded in the Clang front end, LLVM IR will lose the text information of these conflicts, and finallyWizardMerge
will not give suggestions related to them.

###### 0

###### 5

###### 10

###### 15

###### 20

###### 25

###### 30

###### 35

###### 40

###### 45

###### 50

###### 55

###### 60

###### 65

###### 70

###### >

###### Time Consumption (mins)

###### Conflict Pair ID

###### w/ WizardMerge

###### w/o WizardMerge

Fig. 8. Manual merging time w/ and w/oWizardMerge.
The effectiveness of merging w/ and w/oWizardMergeis quantified by time consumption and is shown in Figure 8.
When the time consumption of merging a pair of commits exceeds 70 minutes, we will mark it as "> 70 " and stop
resolving it. In 135 out of 277 conflict scenarios,WizardMergecontributes to reducing version conflict resolution time,
and as the time consumption w/oWizardMergeincreases, the assistance ability ofWizardMergealso becomes more
obvious. Remarkably, for the 205th conflict pair, manual merging w/oWizardMergecosts more than 70 minutes, while
with the help ofWizardMerge, the merging process can be finished in 35 minutes. This exceptional performance is
attributed to the Priority-Oriented Classification module inWizardMerge, which provides categorization and fixes
priority recommendations for manually resolved code blocks. Simultaneously, the Violated DCB Detection module


```
14 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian
```
```
inWizardMergeautomatically identifies code blocks related to conflicts. This feature simplifies developers’ efforts
in analyzing dependencies and avoids the substantial time spent on the manual in-depth exploration of extensive
code space required by Git merge. However,WizardMergemay not optimize the time consumption for all data
points. In 18 out of 277 conflict cases, usingWizardMergecould even increase the burden of conflict resolution. This
occurs when only a small number of conflicts are present and few code blocks are affected, renderingWizardMerge’s
assistance not substantial. Additionally, as mentioned above,WizardMergefails to generate suggestions on comments
or preprocessing instructions.
```
Listing 1. Case study for conflict & violated DCB classification andWizardMerge-produced resolving order assignment.
1 **diff --git a/gfx/layers/ScrollbarData.h b/gfx/layers/ScrollbarData.h**
2 **--- b/gfx/layers/ScrollbarData.h**
3 **+++ a/gfx/layers/ScrollbarData.h**
4 **@@ +38 ,10 -38,9 @@ struct ScrollbarData**
5 *** This constructor is for Thumb layer type.**
6 ***/**
7 **ScrollbarData(ScrollDirection aDirection , float aThumbRatio ,**
8 **<<<<<<< HEAD**
9 **+ OuterCSSCoordaThumbStart,OuterCSSCoordaThumbLength,**
10 **+ OuterCSSCoordaThumbMinLength,boolaThumbIsAsyncDraggable,**
11 **+ OuterCSSCoordaScrollTrackStart,**
12 **+ OuterCSSCoordaScrollTrackLength,uint64_taTargetViewId)**
13 **=======**
14 **- CSSCoordaThumbStart,CSSCoordaThumbLength,**
15 **- boolaThumbIsAsyncDraggable,CSSCoordaScrollTrackStart,**
16 **- CSSCoordaScrollTrackLength,uint64_taTargetViewId)**
17 **>>>>>>> 281943 a7a7564da09bd6076cac0c47a7a62144dc**
18 **: mDirection(Some(aDirection)),**
19 **mScrollbarLayerType(ScrollbarLayerType :: Thumb),**
20 **mThumbRatio(aThumbRatio),**
21 **@@ +65 ,16 -63,16 @@**
22 **public:**
23 **ScrollbarData () = default;**
24
25 **<<<<<<< HEAD**
26 **+staticScrollbarDataCreateForThumb(**
27 **+ ScrollDirectionaDirection,floataThumbRatio,**
28 **+ OuterCSSCoordaThumbStart,**
29 **+ OuterCSSCoordaThumbLength,OuterCSSCoordaThumbMinLength,**
30 **+ boolaThumbIsAsyncDraggable,**
31 **+ OuterCSSCoordaScrollTrackStart,**
32 **+ OuterCSSCoordaScrollTrackLength,uint64_taTargetViewId){**
33 **=======**
34 **- staticScrollbarDataCreateForThumb(ScrollDirectionaDirection,**
35 **- floataThumbRatio,CSSCoordaThumbStart,**
36 **- CSSCoordaThumbLength,**
37 **- boolaThumbIsAsyncDraggable,**
38 **- CSSCoordaScrollTrackStart,**
39 **- CSSCoordaScrollTrackLength,**
40 **- uint64_taTargetViewId){**
41 **>>>>>>> 281943 a7a7564da09bd6076cac0c47a7a62144dc**
42 **return ScrollbarData(aDirection , aThumbRatio , aThumbStart , aThumbLength ,**
43 **+ aThumbMinLength,aThumbIsAsyncDraggable,**
44 **+ aScrollTrackStart,aScrollTrackLength,aTargetViewId);**
45 **- aThumbIsAsyncDraggable,aScrollTrackStart,**
46 **- aScrollTrackLength,aTargetViewId);**
47 **}**
48
49 **@@ MergeGuardian Result**
50 **@@ Group 0:**


```
WizardMerge - Save Us From Merging Without Any Clues 15
```
51 **@@ |**
52 **@@ C-- ScrollbarData xxx/ScrollbarData.h 41-44, 41-**
53 **@@ |**
54 **@@ C-- CreateForThumb xxx/ScrollbarData.h 68-74, 66-**
55 **@@ |**
56 **@@ A-- CreateForThumb xxx/ScrollbarData.h 76-**
57 **@@ |**
58 **@@ B-- (applied) CreateForThumb xxx/ScrollbarData.h 74-**

```
Listing 1 presents a case study illustrating howWizardMergeclassifies the conflicts and violated DCBs and assigns
resolving privileges betweenVA145d3f6 andVB281943a. Initially, both versions introduce distinct modifications to the
ScrollbarDataconstructor, resulting in a merge conflict (lines 8-17). Subsequently, both versions make different alterations
to the parameter types of theCreateForThumbmethod in theScrollbarDataclass, leading to another merge conflict
(lines 25-41). However, Git’s strategy reports these conflicts directly to the developer without providing additional
clues. In contrast,WizardMergeutilizes dependency analysis between definitions to discern that the constructor of
ScrollbarData will be called byCreateForThumb(line 42). Consequently,WizardMergeclassifies the two conflicts into
one group, assigning higher fix priority to lines 8-17 than to lines 25-41.
Additionally, VA and VB present diverse implementations of the function body of theCreateForThumbmethod.
According to the three-way merging algorithm, Git applied the code snippets from VB (lines 45-46) after detecting these
two conflicts. Moreover, within the pair of DCBs represented by lines 43-46, although lines 45-46 are applied without
conflicts, it cannot ensure the constructor ofScrollbarDatareceives the correct parameter types when both conflicts
are present. This is because, if the developer chooses to apply all VA modifications when resolving these conflicts, the
applied lines 45-46 from VB will lack essential parameters,aThumbMinLengthandaThumbIsAsyncDraggabl, used by
ScrollbarDataconstructor and the types ofaThumbStart,aThumbLength,aScrollTrackStart, andaScrollTrackLengthwill
not match theScrollbarDataconstructor prototype. Nevertheless,WizardMergedetects lines 45-46 and its mirror DCB
(i.e., lines 43-44) through its innovative dependency analysis approach, treating them as violated DCBs. It reports them
to developers along with other conflicts. Since lines 43-46 depend not only on lines 8-17 (function call) but also on lines
25-41 (definition of prototype), these two violated DCBs are grouped with the previous two conflicts, with fixed priority
coming after the two conflicts. The resolution suggestions provided byWizardMergeare shown in lines 49-58 of
Listing 1, where items prefixed with C indicate pairs of DCBs with conflicts, and items prefixed with A or B indicate
DCBs from VA or VB considered as violated DCBs.
```
```
Conclusion:Our evaluation confirmsWizardMergecan give suggestions for 72.45% of the conflicts. We
analyze the time expenditures associated with conflict resolution and their corresponding DCBs w/ and w/o the
utilization ofWizardMergeto answeringRQ1. On average,WizardMergedemonstrates a 23.85% reduction
in resolution time. This underscores the efficiency and practicality ofWizardMergein facilitating conflict
resolution, even in the context of large-scale real-world software.
```
```
4.2 Violated DCB Detection
To address bothRQ2andRQ3, our initial step involves establishing the ground truth regarding the number of violated
DCBs by manually merging two code versions using Git merge. Subsequently, we employWizardMergeto conduct
an analysis and document the identified violated DCBs. As not all code segments within a DCB pertain to violated
```

```
16 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian
```
```
Table 2. Violated DCB detection records.CR VArepresents the LoC of CR violated DCB in VA, andCR VBrepresents the LoC of CR
violated DCB in VB. The format𝑛/𝑚indicates that there are𝑚LoCs in total, of whichWizardMergedetects𝑛. Similarly,CU VA
represents the LoC of CU violated DCB in VA, andCU VBrepresents the LoC of CU violated DCB in VB. The format𝑛/𝑚indicates
thatWizardMergedetect𝑚LoCs as CU violated DCB, of which𝑛are confirmed.RecallandPrecisionwith†and‡stand for VA
and VB respectively.
```
```
Dataset CR-VA CR-VB CU-VA CU-VB Recall† Recall‡ Precision† Precision‡
Firefox 151/174 112/123 75/652 98/717 88.30% 91.06% 11.50% 13.67%
Linux Ker-
nel
```
```
107/131 156/192 171/1245 105/905 81.68% 81.25% 13.73% 11.60%
```
```
LLVM 212/316 16/23 N/A N/A 67.09% 69.57% N/A N/A
PHP 92/132 49/139 226/324 213/358 69.70% 35.25% 69.75% 59.50%
MySQL 12274/13100 3171/3469 1750/4597 1307/2427 93.69% 91.41% 38.07% 53.85%
```
```
DCBs, we utilizeLine of Code (LoC)for careful quantification of the violated DCBs. These violated DCBs are further
categorized intoConflict-Related (CR)andConflict-Unrelated (CU)violated DCBs.
4.2.1 Conflict-Related Violated DCBs.In addition to addressing conflict DCBs, developers should reassess the DCBs
automatically applied by Git merge. Resolving one conflict might potentially impact other sets of DCBs. Throughout
our evaluation, we recorded the LoC for DCBs linked to conflicts in both versions. This data reflectsWizardMerge’s
capability in elucidating the correlation between conflicts and violated DCBs.
Table 2 illustrates the CR-violated DCB detection results ofWizardMerge. As VA and VB’s codes are different, the
LoC of CR-violated DCBs in the same diff pair may vary as well. Thus, we documented the LoC from both VA and VB.
According to the evaluation, 167 out of 227 conflict scenarios are confirmed to contain CR-violated DCBs. For most of
the targets,WizardMergesuccessfully detected more than 60% CR-violated DCBs, providing a promising assistive cue
for developers in executing manual fixes and facilitating them in exploring the CR-violated DCBs. Noticeably, although
MySQL datasets have the most LoC of CR-violated DCBs,WizardMerge’s detection recall even reaches 93.69%. For
LLVM and PHP,WizardMergeshowed moderate recalls.
We also delved into the reasons behindWizardMerge’s inability to detect CR-violated DCBs in certain datasets.
Firstly, the Linux Kernel encountered compilation failure with the "O0" option, a crucial setting for preserving debug
information and preventing code optimization. Although "Og" can be used for better debug ability, it is akin to "O1,"
resulting in the elimination of significant debug information [ 14 ]. For instance, inlined functions are inserted at the
invocation location and subsequently removed from the original code. AsWizardMergedepends on the collaboration
of debug information and LLVM IR, the absence of debug information leads to the failure of node generation and DCB
matching. Secondly, some datasets include violated dependencies among local variables within a specific function or
macro definition, whichWizardMerge’s analysis currently does not support.
```
Listing 2. Case study for conflict-related violated DCBs
1 **diff --git a/drivers/net/virtio_net.c b/drivers/net/virtio_net.c**
2 **--- b/drivers/net/virtio_net.c**
3 **+++ a/drivers/net/virtio_net.c**
4 **@@ -126,12 +127 ,9 @@ struct virtnet_stat_desc virtnet_rq_stats_desc [] = {**
5 **#define VIRTNET_SQ_STATS_LEN ARRAY_SIZE(virtnet_sq_stats_desc)**
6 **#define VIRTNET_RQ_STATS_LEN ARRAY_SIZE(virtnet_rq_stats_desc)**
7
8 **<<<<<<< 5c5e0e81202667f9c052edb99699818363b**


```
WizardMerge - Save Us From Merging Without Any Clues 17
```
9 **- structvirtnet_rq_dma{**
10 **- dma_addr_taddr;**
11 **- u32ref;**
12 **- u16len;**
13 **- u16need_sync;**
14 **=======**
15 **+structvirtnet_interrupt_coalesce{**
16 **+ u32max_packets;**
17 **+ u32max_usecs;**
18 **>>>>>>> 1acfe2c1225899eab5ab724c91b7e1eb2881b9ab**
19 **};**
20 **@@ -147,6 +145 ,8 @@ struct send_queue {**
21
22 **struct virtnet_sq_stats stats;**
23
24 **+ structvirtnet_interrupt_coalesceintr_coal;**
25
26 **struct napi_struct napi;**
27 **@@ -164,6 +164 ,8 @@ struct receive_queue {**
28
29 **struct virtnet_rq_stats stats;**
30
31 **+ structvirtnet_interrupt_coalesceintr_coal;**
32
33 **/* Chain pages by the private ptr. */**
34 **@@ -185,12 +187 ,6 @@ struct receive_queue {**
35 **struct xdp_rxq_info xdp_rxq;**
36
37 **- /*Recordthelastdmainfotofree...*/**
38 **- structvirtnet_rq_dma*last_dma;**
39 **- /*Dodmabyself*/**
40 **- booldo_dma;**
41 **};**
42 **@@ -295,10 +292 ,8 @@ struct virtnet_info {**
43 **u32 speed;**
44
45 **/* Interrupt coalescing settings */**
46 **- u32tx_usecs;**
47 **- u32rx_usecs;**
48 **- u32tx_max_packets;**
49 **- u32rx_max_packets;**
50 **+ structvirtnet_interrupt_coalesceintr_coal_tx;**
51 **+ structvirtnet_interrupt_coalesceintr_coal_rx;**

```
Listing 2 presents an intriguing case study revealing conflict-related violated DCBs detected byWizardMerge
betweenVA5c5e0e8 andVB1acfe2c. In VA, the definition forvirtnet_rq_dmais present, whereas at the same position in
VB,virtnet_interrupt_coalesceis defined. Git identifies these versions’ different modifications as a conflict and prompts
manual resolution for developers. Subsequently, Git designates other DCBs as not in conflict and applies them based
on their consistency with the base version’s DCBs. Eventually, the code snippets from lines 24, 31, and 50-51 (from
VA) along with 37-40 (from VB) are applied. Despite the differences between VA and VB in lines 46-51, lines 46-
from VB are consistent with the base version’s snippets. According to the three-way merging strategy, Git prefers to
apply lines 50-51 from VA. In contrast,WizardMergeanalyzes all modified code and discovers that even though the
conflict is highlighted, some related DCBs have been applied without alerting the developers. For instance, applying
lines 37-40 from VB assumes the presence of structvirtnet_rq_dma. Yet, if the developer opts for lines 15-17 as the
conflict resolution, the structure becomes undefined within that context. Similarly, applying lines 24, 31, and 50-51 from
VA assumes the definition of structvirtnet_interrupt_coalesce, while they will cause compilation errors if the developer
```

```
18 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian
```
Listing 3. The definitions of functionEnsureNSSInitialized-
ChromeOrContentare the same in both versions
1 **// security/manager/ssl/nsNSSComponent.cpp**
2 **bool EnsureNSSInitializedChromeOrContent () {**
3 **...**
4 **if (XRE_IsParentProcess ()) {**
5 **// Create nsISupports * via template class nsCOMPtr.**
6 **nsCOMPtr <nsISupports > nss =**
7 **do_GetService(PSM_COMPONENT_CONTRACTID );**
8 **if (!nss) {**
9 **return false;**
10 **}**
11 **initialized = true;**
12 **return true;**
13 **}**
14 **...**
15 **}**

```
Listing 4. Difference between commits d2956560d539 and
a20620423a53 in file xpcom/base/nsCOMPtr.h
1 diff --git a/xpcom/base/nsCOMPtr.h b/xpcom/base/
nsCOMPtr.h
2 --- a/xpcom/base/nsCOMPtr.h
3 +++ b/xpcom/base/nsCOMPtr.h
4 template <class T>
5 class MOZ_IS_REFPTR nsCOMPtr final {
6 ...
7 };
8
9 ...
10
11 +template<>
12 +classMOZ_IS_REFPTRnsCOMPtr<nsISupports>
13 + :privatensCOMPtr_base{
14 + ...
15 +}
```
```
chooses lines 9-13 as the conflict resolution. Consequently,WizardMergeidentifies these code snippets as violated
DCBs in conjunction with the conflict. Furthermore,WizardMergeassigns higher priority to the conflict, signifying
the dependency of other DCBs on the conflict and suggesting developers resolve the conflict before reconsidering these
violated DCBs.
```
```
4.2.2 Conflict-Unrelated Violated DCBs.Throughout this evaluation, we document the LoC for DCBs unrelated to
conflicts in both variant versions reported byWizardMerge. Following this, we manually inspected the identified
DCBs and calculated the number of LoC genuinely violated among them.
The results ofWizardMerge’s detection of CU-violated DCBs are also presented in Table 2. As CU-violated DCBs
are isolated from conflicts,WizardMergedemonstrates the capability to detect them, even in scenarios where it fails to
handle any conflicts. In total,WizardMergedetected CU-violated DCBs in 137 out of 227 conflict scenarios, while no
CU-violated DCBs were found in LLVM. These DCBs, lacking necessary dependent definitions and automatically applied
by Git merge without developer notification, do not lead to conflicts but can result in compile errors or even latent bugs
affecting users at runtime. WhileWizardMergeshowcases its potential in uncovering violated dependency issues, the
evaluation also highlights instances offalse positiveCU-violated DCBs. This arises from the lack of function body
analysis inWizardMerge. According toWizardMerge’s DCB detection algorithm §3.3.2, the relationship between
two DCBs from the same function relies on line numbers, with the latter one forcibly dependent on the former one. If
the two DCBs are applied from different versions,WizardMergemarks them as CU-violated DCBs, even if they are
unrelated. This conservative algorithm may result in more false positives, butWizardMergeprioritizes its application
to prevent overlooking the detection of CU-violated DCBs.
Listing 3 and Listing 4 present another case study whereWizardMergeexplores conflict-unrelated violated DCBs
within Firefox (between versionsVAd295656 andVBa206204), which were overlooked by Git merge. Specifically, in
the filensNSSComponent.cpp, both versions define the functionEnsureNSSInitializedChromeOrContentwith the same
contents. In Listing 3 lines 6-7, a pointer tonsISupportsis created via the template classnsCOMPtr. The definition of
nsCOMPtris found innsCOMPtr.h. However, in addition to the generic template, a template specialization [41] is also
defined fornsISupportsin VB, as shown in lines 11-15 in Listing 4. This specialization ofnsCOMPtrfornsISupports
allows users to employnsCOMPtr<nsISupports>similarly to how they usensISupports*andvoid*, essentially as a
’catch-all’ pointer pointing to any valid [XP]COM interface [ 30 ]. When attempting to mergensCOMPtr.hfrom the two
versions, Git applies the file from VA because the file from VB is identical to the file from the base version. Consequently,
in the functionEnsureNSSInitializedChromeOrContent, the usage ofnsCOMPtr<nsISupports>will follow lines 4-7 in
```

WizardMerge - Save Us From Merging Without Any Clues 19

Listing 4, causingnssto only be able to point to the single[XP]COM-correct nsISupportsinstance within an object. This
inconsistency can lead to a potentialtype confusionbug [ 19 , 21 ]. Since no compilation error occurs after successfully
being applied by Git merge, it would be extremely challenging for developers to notice such a bug, not to mention
identify the root cause. With the assistance ofWizardMerge, the loss of template specialization can be detected in the
merged version, thereby notifying developers to address it manually.

```
Conclusion:We assess the efficacy ofWizardMergein identifying CR-violated and CU-violated DCBs by
quantifying these DCBs in terms of Lines of Code (LoC), addressing bothRQ2andRQ3. The results reveal
thatWizardMergeachieves a detection recall of 80.09% and 73.71% for CR-violated DCBs from both merging
candidates in the five target projects on average. Furthermore, despite attaining precision rates of 33.26% and
34.66% for CU-violated DCBs in the four target projects on average,WizardMergeeffectively reveals deeply
concealed CU-violated DCBs that are overlooked by developers. To underscore the impacts of CR-violated
and CU-violated DCBs, we present case studies that further elucidate the rationale behindWizardMerge’s
noteworthy findings.
```
5 Discussion

In this section, we discuss the present limitations inWizardMerge’s design and explore potential avenues for enhancing
WizardMerge.
Improve Graph Generation.As discussed in §3.3,WizardMergeincorporates three node types and five edge types to
construct the ODG. While it adeptly manages numerous merging scenarios, the evaluation outcomes highlighted in
§4 suggest that this incompleteness could result in an inability to address certain conflicts or violated DCB instances.
For example,WizardMergecannot infer the dependency from a global variable to a function if the global variable
is initialized by the function. This is because, during the compilation process, the initial values of global variables
are established before the execution. In contrast, a function’s return value is accessible only after being executed.
When initializing global variables with a function’s return value, the compiler transforms the function invocation
into initialization code. To ensure the execution of this initialization code before the main program commences, the
compiler positions this code within a specific section of the executable file, commonly denoted as the ".init" segment.
This separation facilitates the execution of these initialization routines prior to the initiation of the main program logic.
We leave updating the features as our future work to adapt to such scenarios.
Assess Code Optimization.WizardMergeconducts dependency analysis based on the compilation metadata. Never-
theless, it overlooks a substantial amount of code information due to compilation optimizations like unused function
elimination and vectorization [ 27 ]. As an interim measure,WizardMergepresently enforces the "O0" option. However,
some projects require compilation optimization for successful building. For instance, the Linux kernel relies on opti-
mization to eliminate redundant code segments and disallows the "O0" option. In the future, we aim to explore how to
adapt our approach to arbitrary optimization options.

6 Related Work

6.1 Merging Strategies

Structured Merging.Mastery [ 45 ] introduces an innovative structured merging algorithm that employs a dual
approach: top-down and bottom-up traversal. It can minimize redundant computations and enable the elegant and


20 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

efficient handling of the code that has been relocated from its original position to another location. Spork [ 23 ] extends
the merging algorithm from the 3DM [ 26 ] to maintain the well-structured code format and language-specific constructs.
To fully retain the original functionality of the merged code, SafeMerge [ 35 ] leverages a verification algorithm for
analyzing distinct edits independently and combining them to achieve a comprehensive demonstration of conflict
freedom.

Semi-structured Merging.JDime [ 4 ] introduces a structured merge approach with auto-tuning that dynamically
adjusts the merge process by switching between unstructured and structured merging based on the presence of conflicts.
FSTMerge [ 5 ] stands as an early example of semi-structured merging, offering a framework for the development
of semi-structured merging. Intellimerge [ 34 ] transforms source code into a program element graph via semantic
analysis, matches vertices with similar semantics, merges matched vertices’ textual content, and ultimately translates
the resulting graph back into source code files.
The above works, however, cannot help the developers with resolving conflicts. On the contrary,WizardMergeis
constructed atop Git merge, furnishing developers with clues to aid manual resolution instead of dictating final merging
decisions.

6.2 Conflict Resolution

Conflict Resolution Assistance.TIPMerge [ 9 ] evaluates the developers’ experience with the key files based on the
project and branch history [ 9 ] to determine the most suitable developers for merging tasks. Automerge [ 44 ] relies on
version space algebra [ 25 ] to represent the program space and implement a ranking mechanism to identify a resolution
within the program space that is highly likely to align with the developer’s expectations. SoManyConflicts [ 33 ] is
implemented by modeling conflicts and their interrelations as a graph and employing classical graph algorithms to
offer recommendations for resolving multiple conflicts more efficiently.

Machine Learning Approaches.MergeBERT [ 38 ] reframes conflict resolution as a classification problem and leverages
token-level three-way differencing and a transformer encoder model to build a neural program merge framework.
Gmerge [ 43 ] investigates the viability of automated merge conflict resolution through k-shot learning utilizing pre-
trained large language models (LLMs), such as GPT-3 [ 16 ]. MergeGen [ 13 ] treats conflict resolution as a generation
task, which can produce new codes that do not exist in input, and produce more flexible combinations.
The essential difference compared with existing assistance systems and machine learning approaches is thatWiz-
ardMergeconsiders all modified code snippets, which further helps developers resolve the violated DCBs in merging
scenarios.

7 Conclusion

We introduceWizardMerge, a novel code-merging assistance system leveraging the merging results of Git and
dependency analysis based on LLVM IR, named definition range, and debug information. Via the dependency analysis,
WizardMergeis able to detect the loss of dependency of the named definitions, therefore providing more accurate
merging order suggestions including conflicts and applied code blocks for the developers. Our evaluation demonstrates
thatWizardMergesignificantly diminishes conflict merging time costs. Beyond addressing conflicts,WizardMerge
provides merging suggestions for most of the code blocks potentially affected by the conflicts. Moreover,Wizard-
Mergeexhibits the capability to identify conflict-unrelated code blocks which should require manual intervention yet
automatically applied by Git.


WizardMerge - Save Us From Merging Without Any Clues 21

References
[1]Paola Accioly, Paulo Borba, and Guilherme Cavalcanti. 2018. Understanding semi-structured merge conflict characteristics in open-source java
projects.Empirical Software Engineering23 (2018), 2051–2085.
[2]Iftekhar Ahmed, Caius Brindescu, Umme Ayda Mannan, Carlos Jensen, and Anita Sarma. 2017. An empirical examination of the relationship
between code smells and merge conflicts. In2017 ACM/IEEE International Symposium on Empirical Software Engineering and Measurement (ESEM).
IEEE, 58–67.
[3]Waad Aldndni, Na Meng, and Francisco Servant. 2023. Automatic prediction of developers’ resolutions for software merge conflicts.Journal of
Systems and Software206 (2023), 111836.
[4]Sven Apel, Olaf Leßenich, and Christian Lengauer. 2012. Structured merge with auto-tuning: balancing precision and performance. InProceedings of
the 27th IEEE/ACM International Conference on Automated Software Engineering. 120–129.
[5]Sven Apel, Jörg Liebig, Benjamin Brandl, Christian Lengauer, and Christian Kästner. 2011. Semistructured merge: rethinking merge in revision
control systems. InProceedings of the 19th ACM SIGSOFT symposium and the 13th European conference on Foundations of software engineering.
190–200.
[6] ApsarasX. 2021. Use LLVM by JavaScript/TypeScript. https://dev.to/apsarasx/use-llvm-by-javascripttypescript-27c3. Accessed: 2024-Feb.
[7] Atlassian. 2022. Git Merge. https://www.atlassian.com/git/tutorials/using-branches/git-merge. Accessed: 2023-Oct.
[8]Guilherme Cavalcanti, Paulo Borba, and Paola Accioly. 2017. Evaluating and improving semistructured merge.Proceedings of the ACM on
Programming Languages1, OOPSLA (2017), 1–27.
[9]Catarina Costa, Jair Figueiredo, Leonardo Murta, and Anita Sarma. 2016. TIPMerge: recommending experts for integrating changes across branches.
InProceedings of the 2016 24th ACM SIGSOFT International Symposium on Foundations of Software Engineering. 523–534.
[10] Cp-Algorithms. 2023. Segment Tree. https://cp-algorithms.com/data_structures/segment_tree.html. Accessed: 2023-Oct.
[11]Elizabeth Dinella, Todd Mytkowicz, Alexey Svyatkovskiy, Christian Bird, Mayur Naik, and Shuvendu Lahiri. 2022. Deepmerge: Learning to merge
programs.IEEE Transactions on Software Engineering49, 4 (2022), 1599–1614.
[12] Jinhao Dong, Qihao Zhu, Zeyu Sun, Yiling Lou, and Dan Hao. [n. d.]. Merge Conflict Resolution: Classification or Generation? ([n. d.]).
[13]Jinhao Dong, Qihao Zhu, Zeyu Sun, Yiling Lou, and Dan Hao. 2023. Merge Conflict Resolution: Classification or Generation?. In2023 38th IEEE/ACM
International Conference on Automated Software Engineering (ASE). IEEE, 1652–1663.
[14] Changbin Du. 2018. kernel hacking: GCC optimization for better debug experience (-Og). https://lwn.net/Articles/753639/. Accessed: 2024-Jan.
[15]Paulo Elias, Heleno de S Campos Junior, Eduardo Ogasawara, and Leonardo Gresta Paulino Murta. 2023. Towards accurate recommendations of
merge conflicts resolution strategies.Information and Software Technology(2023), 107332.
[16] Luciano Floridi and Massimo Chiriatti. 2020. GPT-3: Its nature, scope, limits, and consequences.Minds and Machines30 (2020), 681–694.
[17] Git. 2023. Git. https://git-scm.com/. Accessed: 2023-Oct.
[18] Git. 2023. Git Merge. https://git-scm.com/docs/git-merge. Accessed: 2023-Oct.
[19]Istvan Haller, Yuseok Jeon, Hui Peng, Mathias Payer, Cristiano Giuffrida, Herbert Bos, and Erik van der Kouwe. 2016. TypeSan: Practical Type
Confusion Detection. InProceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security(Vienna, Austria)(CCS ’16).
Association for Computing Machinery, New York, NY, USA, 517–528. https://doi.org/10.1145/2976749.2978405
[20]Kumaran Handy. 2023. The Biggest Codebases in History, as Measured by Lines of Code. https://www.artstation.com/blogs/dioeye/Rn8z/the-
biggest-codebases-in-history-as-measured-by-lines-of-code. Accessed: 2024-Jan.
[21]Yuseok Jeon, Priyam Biswas, Scott Carr, Byoungyoung Lee, and Mathias Payer. 2017. HexType: Efficient Detection of Type Confusion Errors for
C++. InProceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security(Dallas, Texas, USA)(CCS ’17). Association for
Computing Machinery, New York, NY, USA, 2373–2387. https://doi.org/10.1145/3133956.3134062
[22]Jatin Khurana. 2014. GIT merge successfully but introduce a compilation error. https://stackoverflow.com/questions/25270430/git-merge-successfully-
but-introduce-a-compilation-error. Accessed: 2024-Jan.
[23]Simon Larsén, Jean-Rémy Falleri, Benoit Baudry, and Martin Monperrus. 2022. Spork: Structured Merge for Java with Formatting Preservation.IEEE
Transactions on Software Engineering49, 1 (2022), 64–83.
[24] Chris Lattner. 2008. LLVM and Clang: Next generation compiler technology. InThe BSD conference, Vol. 5. 1–20.
[25]Tessa Lau, Steven A Wolfman, Pedro Domingos, and Daniel S Weld. 2003. Programming by demonstration using version space algebra.Machine
Learning53 (2003), 111–156.
[26] Tancred Lindholm. 2004. A three-way merge for XML documents. InProceedings of the 2004 ACM symposium on Document engineering. 1–10.
[27] LLVM. 2023. LLVM’s Analysis and Transform Passes. https://llvm.org/docs/Passes.html. Accessed: 2023-Dec.
[28] LLVM. 2024. The LLVM Compiler Infrastructure. https://llvm.org/. Accessed: 2024-Apr.
[29] Mozilla. 2023. Mercurial Gecko Repositories. https://hg.mozilla.org/. Accessed: 2023-Dec.
[30]Mozilla. 2024. Gecko-dev: nsCOMPtr.h. https://github.com/mozilla/gecko-dev/blob/a20620423a5363cf7afdac81e062bc687c29366a/xpcom/base/
nsCOMPtr.h Accessed: 2024-01-29.
[31] MySQL. 2024. MySQL. https://dev.mysql.com/. Accessed: 2024-Apr.
[32]Yuichi Nishimura and Katsuhisa Maruyama. 2016. Supporting merge conflict resolution by using fine-grained code change history. In2016 IEEE 23rd
International Conference on Software Analysis, Evolution, and Reengineering (SANER), Vol. 1. IEEE, 661–664.


22 Qingyu Zhang, Junzhe Li, Jiayi Lin, Jie Ding, Lanteng Lin, and Chenxiong Qian

[33]Bo Shen, Wei Zhang, Ailun Yu, Yifan Shi, Haiyan Zhao, and Zhi Jin. 2021. SoManyConflicts: Resolve many merge conflicts interactively and
systematically. In2021 36th IEEE/ACM International Conference on Automated Software Engineering (ASE). IEEE, 1291–1295.
[34]Bo Shen, Wei Zhang, Haiyan Zhao, Guangtai Liang, Zhi Jin, and Qianxiang Wang. 2019. IntelliMerge: a refactoring-aware software merging
technique.Proceedings of the ACM on Programming Languages3, OOPSLA (2019), 1–28.
[35]Marcelo Sousa, Isil Dillig, and Shuvendu K Lahiri. 2018. Verified three-way program merge.Proceedings of the ACM on Programming Languages2,
OOPSLA (2018), 1–29.
[36]Calvin Spealman. 2018. When a Clean Merge is Wrong. https://www.caktusgroup.com/blog/2018/03/19/when-clean-merge-wrong/. Accessed:
2024-Jan.
[37]Alexey Stepanov. 2020. What’s the difference between Kotlin/Native and Java bytecode with compile through LLVM frontend? https://stackoverflow.
com/questions/60580188/whats-the-difference-between-kotlin-native-and-java-bytecode-with-compile-throu. Accessed: 2024-Feb.
[38]Alexey Svyatkovskiy, Sarah Fakhoury, Negar Ghorbani, Todd Mytkowicz, Elizabeth Dinella, Christian Bird, Jinu Jang, Neel Sundaresan, and
Shuvendu K Lahiri. 2022. Program merge conflict resolution via neural transformers. InProceedings of the 30th ACM Joint European Software
Engineering Conference and Symposium on the Foundations of Software Engineering. 822–833.
[39] Robert Tarjan. 1972. Depth-first search and linear graph algorithms.SIAM journal on computing1, 2 (1972), 146–160.
[40] Linus Torvalds. 2023. Linux Kernel Official Repository. https://github.com/torvalds/linux. Accessed: 2023-Oct.
[41] David Vandevoorde and Nicolai M Josuttis. 2002.C++ templates: The complete guide, portable documents. Addison-Wesley Professional.
[42] Alexis Määttä Vinkler. 2023. The Magic of 3-Way Merge. https://blog.git-init.com/the-magic-of-3-way-merge/. Accessed: 2023-Oct.
[43]Jialu Zhang, Todd Mytkowicz, Mike Kaufman, Ruzica Piskac, and Shuvendu K Lahiri. 2022. Using pre-trained language models to resolve textual
and semantic merge conflicts (experience paper). InProceedings of the 31st ACM SIGSOFT International Symposium on Software Testing and Analysis.
77–88.
[44]Fengmin Zhu and Fei He. 2018. Conflict resolution for structured merge via version space algebra.Proceedings of the ACM on Programming
Languages2, OOPSLA (2018), 1–25.
[45]Fengmin Zhu, Xingyu Xie, Dongyu Feng, Na Meng, and Fei He. 2022. Mastery: Shifted-Code-Aware Structured Merging. InInternational Symposium
on Dependable Software Engineering: Theories, Tools, and Applications. Springer, 70–87.


