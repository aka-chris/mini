# mini-gunrock
This little project is the results of my study of moderngpu 2.0 and my attempt to create a minimalism-style graph processing library for the GPU. The reasons of creating this project are:
- Gunrock main project is getting huge and it is difficult for new developers to get the core idea of our original purpose: giving graph processing library on the GPU both programmability and performance.
- The integration with Multi-GPU and template-based design make it difficult to process micro-benchmarks for single workload mapping strategies and operations.
- The Gunrock main project uses a mix of C++03, C++11, C99 and old-fashioned CUDA programming style, an alternative of refactoring such a large project is to build a small project with minimal components so that we can quickly try out new ideas.

# Introduction
mini-gunrock's core components are graph traversal operators that based on moderngpu 2.0 transforms. The use of moderngpu 2.0 makes the code size small without losing performance. Here is a brief introduction of all the components in mini-gunrock.
- **graph_device_t** contains CSR, CSC, and edge list presentations, it is the main data structure for storing our graph topology and node/edge values. It also contains auxiliary data, such as *d_scanned_row_offsets*, to be used by traversal operators.
- **frontier_t** is the input and output for all traversal operators. It maps to Gunrock's frontier_queue using 1DArray.
- **problem_t** is the base class for different graph problems. It contains a shared_ptr to graph_device_t called *gslice*. Each graph primitive will derive this class and define their own problem data structure which contains per-node/per-edge data.
- **filter and advance** map to Gunrock's filter and advance operators. Underneath, they both use moderngpu 2.0's transforms (transform_compact for filter, and transform_scan + transform_lbs for advance). Currently I just implemented a baseline implementation which equals to Gunrock's LB strategy without idempotence capability. I will gradually add more features as I explore more workload mapping strategies.

`mini/gunrock/tests/bfs/test_bfs.cu` shows the power of mini-gunrock. After loading graph and setting up frontier and problem, the actual algorithm part only contains 8 lines of code. It is a truly data-centric framework and basically achieved our original idea of "the flow of frontier between multiple operators".

# TODOS (with no perticular orders)
- Add dynamic group workload mapping strategy and filter with flexible uniquification from Duane Merrill's PPoPP'12 paper.
- Add merge-based advance-reduce operator based on Duane Merrill's SpMV idea.
- Add a pure compute operator with no filter (maps to Gunrock's bypass_filter).
- Add launch_box and restrict settings to further improve the performance.
- Generalize advance with automatically push and pull configurations as in Ligra.
- Add the batch-intersection operator.
- Add more graph primitives and their CPU validation code.

# To Gunrock Team
As I'm still actively working on this project, you will see more changes in the coming up months, please try it out, provide feedbacks, and contribute to the code if you want. Hope this project can serve as both a quick experiment playground for new ideas and also a better way to get familiar with our data-centric framework.

YZH
