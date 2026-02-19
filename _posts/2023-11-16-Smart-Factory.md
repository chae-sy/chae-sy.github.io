---
title: "[Project] Smart Factory Capstone Design : Rotor Fault Diagnosis Method Based on 1D-CNN LSTM"
date: 2023-11-16
---
This paper presents a fault diagnosis method for rotating machinery using a 1D-CNN LSTM architecture. The proposed model effectively captures temporal and frequency-domain features from vibration data, achieving high diagnostic accuracies for various fault types. The study emphasizes the importance of early fault detection in manufacturing environments to prevent downtime and enhance efficiency.

## Anomaly Detection Based on 1D-CNN LSTM for Rotating Machines

**Dae-Hee Lee¹, Se-Won Kim², Jung-Min Yoo³, Ye-Won Jeong⁴, Seo-Yoon Chae⁵, Chae-Gyu Lee⁶, Hyun-Seung Choo⁷, Jong-Pil Jeong⁸**

¹Department of Electrical and Computer Engineering, Sungkyunkwan University

²School of Civil, Architectural and Environmental System Engineering, Sungkyunkwan University

³School of Civil, Architectural and Environmental System Engineering, Sungkyunkwan University

⁴Department of Systems Management Engineering, Sungkyunkwan University

⁵School of Electrical and Computer Engineering, Sungkyunkwan University

⁶Department of Smart Factory Convergence, Sungkyunkwan University (Co-corresponding author)

⁷Department of Electrical and Computer Engineering, Sungkyunkwan University (Co-corresponding author)

⁸Department of Smart Factory Convergence, Sungkyunkwan University (Co-corresponding author)

---

## Abstract

The manufacturing industry is rapidly evolving in the era of the Fourth Industrial Revolution, emphasizing large-scale data, high speed, and low power consumption. Consequently, the importance of rotating machinery has significantly increased. This paper proposes a **fault diagnosis method for rotating machines based on 1D-CNN LSTM (One-Dimensional Convolutional Neural Network Long Short-Term Memory)**. The proposed deep learning model is capable of recognizing complex patterns and making accurate predictions from large datasets. Experimental results demonstrate fault diagnosis accuracies of **95.6%, 71.4%, and 97.4%** for different fault types. The proposed model enables early detection of faults, preventing production line downtime and enhancing manufacturing efficiency and competitiveness.

**Keywords:** Anomaly Detection, 1D-CNN LSTM, Rotating Machine, Deep Learning

---

## I. Introduction

With the advent of the Fourth Industrial Revolution, manufacturing systems are evolving toward large-scale, high-speed, and energy-efficient operations. Rotating machinery, as a core component of industrial processes, has become increasingly critical. Any malfunction in such machinery can severely disrupt the entire production process, leading to significant economic losses. Early detection of faults through deep learning can minimize these losses and improve production efficiency, providing both industrial and economic benefits.

Recent studies have explored deep learning-based approaches such as **DNNs**, **CNNs**, and **RNNs** for rotating machinery fault diagnosis. Since deep learning models can learn complex patterns from large datasets, it becomes feasible to detect early-stage anomalies using data such as noise and vibration. This paper proposes a fault diagnosis method based on **1D-CNN LSTM**, targeting imbalance and looseness conditions. The method aims to prevent unexpected halts in the production process, thereby improving operational efficiency and reliability.

---

## II. Methodology

This study utilizes the **rotating machinery fault-type AI dataset** provided by **KAMP (Korea AI Manufacturing Platform)**.

Figure 1 shows the **Rotor Testbed**, implemented using the **ERA (Educational Rotor Application) Test Station** from **Signallink**. The testbed consists of three primary modules: a **Controller Module**, a **BLDC Motor (Brushless DC Motor)**, and **Disks**.

The **Controller Module** adjusts the rotational speed (RPM) of the disks; in this experiment, vibration data were collected at **1,500 RPM**. The BLDC motor enables fine-grained speed control without mechanical brushes. The disk section contains 36 bolt holes, and mass imbalance was introduced by fixing a single bolt at a 270° position.

![image.png](/images/2023-11-16/image.png)

**Figure 1. Rotor Testbed**

The proposed **1D-CNN LSTM-based fault diagnosis system** is illustrated in Figure 2.

The **1D-CNN** effectively captures temporal and frequency-domain features in time-series data, while the **LSTM** network learns long-term dependencies inherent in such sequences. The combination of these two architectures allows efficient extraction and interpretation of temporal patterns in rotating machinery vibration data.

![image.png](/images/2023-11-16/image%201.png)

**Figure 2. 1D-CNN LSTM Architecture**

---

## III. Experimental Results

Four categories of experimental datasets were used:

1. **Normal condition (Normal)**
2. **Mass imbalance (Unbalance)** — created by attaching a single bolt to the disk
3. **Mechanical looseness (Mechanical Looseness)** — simulated by partially loosening a bolt
4. **Combined fault (Unbalance + Mechanical Looseness)**

Each dataset contained vibration signals collected over **140 seconds**. Preprocessing steps included **linear interpolation, filtering, and normalization**. The data were split into **240,000 training**, **80,000 validation**, and **80,000 test samples**.

Instead of performing multi-class classification, binary classification was conducted for each fault type. Binary classification simplifies learning and reduces training time—crucial for real-time applications in manufacturing environments where rapid fault detection is essential. Moreover, since mass imbalance is the most frequent fault type in practice, it was emphasized in the modeling process.

The results are visualized in Figures 3–5 using confusion matrices. The proposed method achieved diagnostic accuracies of **95.6%, 71.4%, and 97.4%** for the three fault types, respectively.

![image.png](/images/2023-11-16/image%202.png)

**Figure 3. Normal vs. Unbalance**

![image.png](/images/2023-11-16/image%203.png)

**Figure 4. Normal vs. Mechanical Looseness**

![image.png](/images/2023-11-16/image%204.png)

**Figure 5. Normal vs. Unbalance and Mechanical Looseness**

---

## IV. Conclusion

This paper presented a **1D-CNN LSTM-based fault diagnosis system** for rotating machinery using the KAMP fault-type AI dataset. The proposed architecture effectively processes time-series vibration signals by leveraging 1D-CNN for spatial pattern extraction and LSTM for learning long-term dependencies.

Experimental results demonstrated accurate classification of fault conditions, with diagnostic accuracies of **95.6%, 71.4%, and 97.4%**. The model is particularly effective in detecting frequent real-world issues such as mass imbalance, making it promising for industrial deployment. The relatively lower accuracy in the unbalance dataset (71.4%) is attributed to the small amplitude variations in the vibration signals generated under this condition. Future work will extend this study to **multi-class classification** and further model refinement.

---

### Acknowledgment

This research was supported by the **LINC 3.0 Project**, funded by the **Ministry of Education** and the **National Research Foundation of Korea (NRF)**.

---

### References

[1] Y. Huang, S. Sun, X. Duan, and Z. Chen, “A Study on Deep Neural Networks Framework,” *IEEE Advanced Information Management, Communicates, Electronic and Automation Control Conference (IMCEC)*, pp. 1519–1522, 2016.

[2] K. O'Shea and R. Nash, “An Introduction to Convolutional Neural Networks,” *arXiv preprint arXiv:1511.08458*, 2015.

[3] W. Zaremba, I. Sutskever, and O. Vinyals, “Recurrent Neural Network Regularization,” *arXiv preprint arXiv:1409.2329*, 2014.

[4] W. Chung, S. Lee, T. Chung, and J. Lee, “Fault Diagnosis of a Pump by Using Vibrational Signals,” *Proceedings of the KSME Conference*, pp. 590–595, 2001.

[5] D. Lee, H. Choo, and J. Jeong, “Anomaly Detection Based on 1D-CNN-LSTM Auto-Encoder for Bearing Data,” *WSEAS Transactions on Information Science and Applications*, Vol. 20, pp. 1–6, 2023.

[6] S. Kiranyaz *et al.*, “1D Convolutional Neural Networks and Applications: A Survey,” *Mechanical Systems and Signal Processing*, Vol. 151, 2021.

[7] R. C. Staudemeyer and E. R. Morris, “Understanding LSTM – A Tutorial into Long Short-Term Memory Recurrent Neural Networks,” *arXiv preprint arXiv:1909.09586*, 2019.

[8] Korea AI Manufacturing Platform (KAMP), “Rotating Machine Fault-Type AI Dataset,” 2021. [https://www.kamp-ai.kr](https://www.kamp-ai.kr/)

[9] Signallink, “Educational Rotor Application Test Station,” [http://www.signallink.co.kr](http://www.signallink.co.kr/)