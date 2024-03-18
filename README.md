# Chromagram Self-Similarity Matrix Segmentation and Greedy Algorithm Implementations
Repository containing implementations and adaptations of the methodology presented by Mauch, Noland, and Dixon (2009) at the 10th International Society for Music Information Retrieval Conference (ISMIR) [1] for segmenting music by analyzing repeating structures from beat-synced chromagram self-similarity matrices (SSM).

## Introduction
This project focuses on replicating and extending the automated chromagram Self-Similarity Matrix (SSM) segmentation process of [1] to explore and improve upon existing methods of automatic music segmentation. 

## Background
Chromagrams and SSMs play a pivotal role in music information retrieval, enabling the detailed analysis of musical structures. This project delves into the nuances of these concepts, aiming to leverage their potential for automated music segmentation.

## Methodology

### Generating Beat and Meter Locations
The `generate_beats_and_meters` function calculates the beat and meter locations within a song, from an array of beat frames, tempo, and an assumed time signature of 4 beats per meter. It takes into consideration the song's tempo, duration, and time signature.

### Key-Invariant Chromagram Calculation
We implement the Krumhansl-Schmuckler algorithm to calculate a Key-Invariant (KI) chromagram. We choose to use KI chromagrams over a standard Constant-Q Transform (CQT) chromagram for 2 reasons:
1. Key-invariance allows for comparisons and pattern analyses between songs/segments across different keys. The KI chromagram, by applying a key-finding algorithm and shifting the chroma values accordingly, normalizes these differences, allowing for a more accurate comparison of musical segments regardless of their original key. Although the scope of this project doesn't entail making cross key comparisons, it has the capability to do so. 
2. KI chromagrams inadvertently reduces the variation caused by key changes, which could be considered "noise" in the context of structural analysis, by aligning chroma vectors to a normalized key.

### Beat-Synchronous KI Chromagram 
We synchronize the KI chroma features with the detected beats. For each beat interval, we aggregate the chroma vectors that fall within it using the median to minimize the impact of outliers. This results in a chromagram where each column corresponds to a beat interval, ensuring that the harmonic content is aligned with the musical rhythm.
![beat-sync-ki-chroma](/images/beat-sync-ki-chroma.webp)

### Constructing the Self-Similarity Matrix (SSM)
- Before computing pairwise distances, we transpose the chromagram matrix. This step is essential to ensure that the resulting SSM effectively represents the correlation of harmonic content across the time axis. It allows us to compare each time frame (or beat-synced frame) against every other, rather than compare each pitch class. 
- The size of the SSM is determined by the number of beat-synced time frames (i.e. beats) in the transposed chroma feature matrix. Each axis of the SSM corresponds to the beat index in the chromagram, with the cell values representing the similarity between these frames.
- After calculating the SSM, we take the absolute values of the matrix because it makes the SSM more interpretable and useful for identifying repeating musical structures: values closer to 1 indicate a high degree of similarity between segments, while values closer to 0 suggest dissimilarity. 

#### Median Filtered SSM
Aligning with the methods of Mauch, Noland, and Dixon [1], we then apply a diagonal median filter to the normalized matrix with `kernel_size = 5` (an odd number that conveniently happens to cover slightly over a bar of music). Diagonal median filtering helps in smoothing and minimizing the impact of minor variations or noise in the chromagram, leading to a cleaner representation of similarity across time frames. For visualizations, it enhances the visibility of significant repeating patterns by diminishing the influence of spurious correlations.

#### Path-enhanced SSM
For empirical testing, we employ path enhancement techniques on the SSM. Path enhancement serves to make diagonal paths, which represent repeating musical segments, more pronounced. This is particularly beneficial for uncovering temporal structures and patterns that recur over time, making it easier to identify repeating segments in pieces with complex structures or subtle repetitions.

We use `librosa.segment.path_enhance()` for path enhancement, inspired by the multi-angle path enhancement approach detailed by Müller and Kurth [2]. Unlike their method, which models tempo differences by re-sampling underlying features before generating the SSM, our approach directly enhances continuity along diagonal paths within the existing similarity matrix. This direct enhancement of the SSM facilitates the identification of repeating structures without altering the chromagram data.

#### Comparing Median Filter vs. Path Enhanced SSM for Segmentation
One of the goals of this project to compare the effectiveness of median filtering versus path-enhanced SSM in facilitating accurate music segmentation. By evaluating both approaches, we aim to determine which method and parameterse leads to more accurate and meaningful segmentation of repeating musical segments.

![ssm-types](/images/ssm-types.webp)

### Finding Repetitions in Self-Similarity Matrices
The `find_repetitions` function is designed to identify repeating segments within a piece of music by analyzing its SSM. The code is a custom implementation adapted from the methodology presented by [Mauch, Noland, and Dixon (2009)] at the 10th International Society for Music Information Retrieval Conference (ISMIR).

### Greedy Segment Selection Algorithms
This section describes the original and two alternative greedy algorithms implemented for segment selection, highlighting their methodologies and differences.

## Implementation

### Beat and Chromagram Extraction
Code snippets and explanations detail the process of extracting beat-synced chromagrams, essential for accurate SSM construction.

### SSM Construction and Enhancement
Steps for constructing the SSM and applying enhancements to improve its utility for segment analysis.

### Greedy Algorithms Implementation
A comprehensive look at the implementation of the three greedy algorithms, including the logic behind each and the rationale for their comparisons.

## Results and Visualization

### Visualization Techniques
Techniques and code for visualizing the outcomes of each greedy algorithm on both median-filtered and path-enhanced SSMs of beat-synchronous key-invariant chromagrams.

### Comparative Analysis
A discussion on the performance comparisons of the greedy algorithms, offering insights into their effectiveness and applicability.

## Discussion
A reflection on the results, challenges faced, and the implications of this project's findings on the field of automated music analysis.

## Conclusion and Future Work
Summarizing the project's contributions to automated chromagram segmentation and suggesting directions for future research and development.

Reference:
[1] Mauch, M., Noland, K., & Dixon, S. (2009). Using Musical Structure to Enhance Automatic Chord Transcription. In Proceedings of the 10th International Society for Music Information Retrieval Conference (ISMIR 2009).
[2] Müller, Meinard and Frank Kurth. "Enhancing similarity matrices for music audio analysis." 2006 IEEE International Conference on Acoustics Speech and Signal Processing Proceedings. Vol. 5. IEEE, 2006.
