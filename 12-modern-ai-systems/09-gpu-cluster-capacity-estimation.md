# GPU Cluster Capacity & VRAM Estimation

Sizing GPU infrastructure for LLM serving requires calculating memory requirements across model parameters, KV cache, activations, and compute TFLOPs.

---

## 1. Total GPU VRAM Sizing Formula

$$\text{Total VRAM Required} = \text{Model Weights Memory} + \text{KV Cache Memory} + \text{Activations \& CUDA Overhead}$$

### A. Model Weights Memory
$$\text{Weights Memory} = \text{Parameter Count} \times \text{Bytes Per Param}$$
- **Llama-3-70B FP16 ($2\text{ bytes}$)**: $70\text{B} \times 2 = \mathbf{140\text{ GB VRAM}}$.
- **Llama-3-70B INT4 Quantized ($0.5\text{ bytes}$)**: $70\text{B} \times 0.5 = \mathbf{35\text{ GB VRAM}}$.

### B. KV Cache Memory (Batch Size $B$, Context Length $L$)
$$\text{KV Cache VRAM} = B \times L \times 320\text{ KB (for 70B GQA)}$$
- For $B = 64$ concurrent streams and $L = 4096$ tokens:
  $$\text{KV Cache} = 64 \times 4096 \times 320\text{ KB} \approx \mathbf{84\text{ GB VRAM}}.$$

### C. Total Node Hardware Recommendation
- **Total VRAM Needed**: $140\text{ GB (Weights)} + 84\text{ GB (KV)} + 16\text{ GB (Overhead)} = \mathbf{240\text{ GB VRAM}}$.
- **Server Selection**: $8 \times \text{NVIDIA A100 (80GB)}$ or $4 \times \text{H100 (80GB)}$ with **Tensor Parallelism ($TP = 4$ or $8$)** over NVLink.
