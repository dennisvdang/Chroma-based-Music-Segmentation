# Chromagram Self-Similarity Matrix Segmentation and Greedy Algorithm Implementations
Repository containing implementations and adaptations of the methodology presented by Mauch, Noland, and Dixon (2009) at the 10th International Society for Music Information Retrieval Conference (ISMIR) [1] for segmenting music by analyzing repeating structures from beat-synced chromagram self-similarity matrices (SSM).

![repeating-segments](/images/repeating-segments.webp)

## Introduction
This project focuses on replicating and extending the automated chromagram Self-Similarity Matrix (SSM) segmentation process of [1] to explore and improve upon existing methods of automatic music segmentation. 

## Methodology

### Key-Invariant Chromagram Calculation
We implement the Krumhansl-Schmuckler algorithm to calculate a Key-Invariant (KI) chromagram. We choose to use KI chromagrams over a standard Constant-Q Transform (CQT) chromagram for 2 reasons:
1. Key-invariance allows for comparisons and pattern analyses between songs/segments across different keys. The KI chromagram, by applying a key-finding algorithm and shifting the chroma values accordingly, normalizes these differences, allowing for a more accurate comparison of musical segments regardless of their original key. Although the scope of this project doesn't entail making cross key comparisons, it has the capability to do so. 
2. KI chromagrams inadvertently reduces the variation caused by key changes, which could be considered "noise" in the context of structural analysis, by aligning chroma vectors to a normalized key.

### Beat-Synchronous Key-Invariant Chromagram 
- Beat onsets and tempo are extracted and used to calculate the frame locations of beats and meters.
- For each beat interval, we aggregate the chroma vectors that fall within it using the median (median vs. mean aggregation seems to be negligible here). This results in a chromagram where each column corresponds to a beat interval, ensuring that the harmonic content is aligned with the musical rhythm.
![beat-sync-ki-chroma](/images/beat-sync-ki-chroma.webp)

### Constructing the Self-Similarity Matrix (SSM)
- Before computing pairwise distances, we transpose the chromagram matrix. This step is essential to ensure that the resulting SSM effectively represents the correlation of harmonic content across the time axis. It allows us to compare each time frame (or beat-synced frame) against every other, rather than compare each pitch class. 
- We compute the Pearson correlation coefficients between every pair of chroma vectors to construct a beat-wise self-similarity matrix \(R = (r_{ij})\) for the entire song. This approach is analogous to utilizing a matrix of cosine distances as described by Ong [3]. In the similarity matrix, parallel diagonal lines signal the presence of repeated sections throughout the song. 
- The size of the SSM is determined by the number of beat-synced time frames (i.e. beats) in the transposed chroma feature matrix. Each axis of the SSM corresponds to the beat index in the chromagram, with the cell values representing the similarity between these frames.
- After calculating the SSM, we take the absolute values of the matrix because it makes the SSM more interpretable and useful for identifying repeating musical structures: values closer to 1 indicate a high degree of similarity between segments, while values closer to 0 suggest dissimilarity. 

#### Median Filtered SSM
- To mitigate the effects of short-term noise or deviations, and align with the methods of [1], we apply a median filter with a length of 5 (just over one bar's length) diagonally across the similarity matrix. This processing step ensures that minor deviations are tolerable.

#### Path-enhanced SSM
- One of the goals of this project to compare the effectiveness of median filtering versus path-enhanced SSM in facilitating accurate music segmentation. Path enhancement serves to make diagonal paths, which represent repeating musical segments, more pronounced. This is particularly beneficial for uncovering temporal structures and patterns that recur over time, making it easier to identify repeating segments in pieces with complex structures or subtle repetitions.

- We use `librosa.segment.path_enhance()` for path enhancement, inspired by the multi-angle path enhancement approach detailed by Müller and Kurth [2]. Unlike their method, which models tempo differences by re-sampling underlying features before generating the SSM, our approach directly enhances continuity along diagonal paths within the existing similarity matrix. This direct enhancement of the SSM facilitates the identification of repeating structures without altering the chromagram data.

- By evaluating both approaches, we aim to determine which method and parameterse leads to more accurate and meaningful segmentation of repeating musical segments.

![ssm-types](/images/ssm-types.webp)

### Finding Repetitions in Self-Similarity Matrices
To analyze a song for repeated musical segments, we compare segments of the music that start at different beats but are of equal length. We're specifically looking for segments that are either exactly the same or very similar to each other.

**Segment Comparison**: For each pair of segments starting at beats $i$ and $j$, we examine their corresponding elements (or beats) to assess how similar they are. This evaluation generates a series of values, recorded in the Self-Similarity Matrix as Pearson correlation coefficients, which quantify the similarity over the full span of the segments under comparison.

$$D_{i,j,l} = \left( r_{i,j}, r_{i+1,j+1}, \ldots, r_{i+l,j+l} \right)$$

**Identifying Perfect and Near-Perfect Matches**: Ideally, if two segments are exactly the same, all the comparison values would be ones, indicating a perfect match. However, not all repetitions are perfect due to variations in performance, recording, or intended musical changes. Thus, near-perfect matches are allowed and are defined using two parameters, $p$ and $t_s$, where we used the same parameter values of $p = 0.1$ and $t_s$ as the original study.

**Handling overlaps**: Once segments that meet or exceed the similarity threshold are identified, they are added to a list of repetitions. If there's any overlap between identified repetitions, only the ones with the highest similarity (according to the set thresholds) are kept.

**Searching for Repetitions**: A search for repetitions is conducted across all diagonals in the matrix, spanning a variety of segment lengths. Employing the same parameters as [1], we designate a minimum segment length of $m_1 = 12$ beats and a maximum of $m_M = 128$ beats, creating a vast search space. To optimize our search, we only consider beats with a correlation $r$ exceeding a threshold $t_r$, and we presuppose that segment durations are quantized to multiples of four beats. We employed a threshold of $t_r = 0.65$ similar to [1]. Future endeavors aim to refine $t_r$ based on empirical data. The original study further narrows the search space by only allowing segments that start at the beginning of bars, calculated using a harmony change likelihood function and beat detection function, which is outside the scope of this project.

**Greedy Algorithm for Selection**: 
- We replicate the greedy algorithm used to identify the most noteworthy repeating segments. This process starts by selecting the repetition set whose product of `length` and `similarity score` is the highest, then continues to evaluate all segments under this criterion. An exception to this selection process is made when a sub-segment occurs more frequently within longer ones; the subsegment is given precedence in these cases. While the precise mechanics of the original greedy algorithm as described by [1] remain somewhat open to interpretation due to linguistic ambiguity, our approach has been to closely emulate the described methodology while also developing two additional algorithms to facilitate downstream analyses. 
- We created an additional greedy algorithm that aims to maximize the total length of the music that is covered by identified repeating patterns. 
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
[3] Ong, B. S. (2006). "Structural Analysis and Segmentation of Music Signals." *Universitat Pompeu Fabra*.
