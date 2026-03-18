# Deep Learning Theory & Practice -- Problem Bank, Walkthroughs & Calculations

---

## Problem 1: Transformers vs RNNs -- A Complete Analysis

### Key Concepts
- **RNN sequential bottleneck**: h_t = f(h_{t-1}, x_t). Information from token 1 must pass through every intermediate hidden state to reach token N.
- **Gradient path length**: In RNNs, the gradient path from loss at position N to parameters affecting position 1 is O(N). In Transformers, it is O(1) via direct attention connections.
- **Parallelism**: RNN training is sequential across time steps. Transformer attention is a matrix multiplication -- fully parallelizable.

### Walkthrough

**1. Sequential Dependency Problem**
- RNN: h_t depends on h_{t-1}. Cannot compute h_100 without first computing h_1 through h_99.
- Transformer: all positions computed simultaneously. Attention scores for all pairs computed in one matrix multiply: Attention = softmax(QK^T / sqrt(d_k)) * V.
- Practical impact: Transformers train 10-100x faster on modern GPU hardware for the same sequence length.

**2. Long-Range Dependencies**
- LSTM mitigates vanishing gradients via cell state highway, but in practice, effective memory is limited to a few hundred tokens.
- Transformer self-attention: token at position 1 directly attends to token at position 1000. The gradient flows through a single attention weight, not through 999 intermediate states.
- Evidence: Transformers consistently outperform LSTMs on tasks requiring long-range reasoning (document QA, long-form generation).

**3. Trade-offs**
- Attention is O(n^2) in sequence length (memory and compute). RNNs are O(n).
- For very long sequences (>10K tokens), this becomes prohibitive. Solutions: sparse attention (BigBird), linear attention (Performer), sliding window (Longformer), FlashAttention (hardware-optimized).
- RNNs have constant memory at inference (just the hidden state). Transformers need the full KV cache. This matters for streaming applications.
- Recent work (Mamba, RWKV) revisits recurrent architectures with selective state spaces, achieving Transformer-like quality with linear-time inference.

**4. Position Information**
- RNNs get position implicitly from sequential processing order.
- Transformers are permutation-invariant without positional encoding. Must add explicit position information.
- Options: sinusoidal (fixed, generalizes to unseen lengths), learned (data-adaptive), rotary (RoPE, combines benefits, used in modern LLMs).

---

## Problem 2: Training a Large Language Model

### Key Calculations
- **Model size**: 7B params * 4 bytes (fp32) = 28 GB. In bf16: 14 GB.
- **Optimizer state (Adam)**: 2 * 7B * 4 bytes (fp32 first and second moments) = 56 GB.
- **Gradients**: 7B * 2 bytes (bf16) = 14 GB.
- **Total per-GPU without parallelism**: 14 + 56 + 14 = 84 GB (does not fit on one 80 GB A100, and we have not counted activations yet).

### Walkthrough

**1. Memory Optimization**
- Mixed precision (bf16): model weights and activations in bf16. Loss computation in fp32 for numerical stability.
- ZeRO Stage 1: shard optimizer states across data-parallel ranks. With 8 GPUs: 56 GB / 8 = 7 GB per GPU.
- ZeRO Stage 2: also shard gradients. With 8 GPUs: (56 + 14) GB / 8 = 8.75 GB per GPU.
- Activation checkpointing: do not store intermediate activations. Recompute during backward pass. Trades ~33% more compute for ~60% less activation memory.

**2. Parallelism Strategies**
- Data parallelism: replicate model on each GPU, shard data. All-reduce gradients after each step. Works well up to ~32-64 GPUs before communication becomes bottleneck.
- Tensor parallelism: split individual layers across GPUs (e.g., split attention heads, split MLP columns). Requires fast interconnect (NVLink). Typically within a node (4-8 GPUs).
- Pipeline parallelism: split model layers across GPUs. GPU 1 has layers 1-16, GPU 2 has layers 17-32. Micro-batching to reduce bubble overhead.
- Typical setup for 7B: tensor parallelism (TP=4) within node + data parallelism across nodes.

**3. Training Stability**
- Learning rate: warmup over 1000-2000 steps to peak (1e-4 to 3e-4), then cosine decay to 10% of peak.
- Gradient clipping: max norm of 1.0 to prevent loss spikes.
- Weight decay: 0.1 with AdamW (decoupled weight decay, not L2 regularization).
- Monitor: loss curve, gradient norms, learning rate, throughput (tokens/sec), GPU utilization.

**4. Data Pipeline**
- Tokenize entire corpus upfront. Store as binary files with memory-mapped access.
- Shuffle at the document level, not token level (preserve document coherence).
- Pack multiple short documents into one sequence with document separator tokens.
- Data loading must not be the bottleneck: prefetch, multiple workers, pin memory.

**5. Evaluation**
- Perplexity on held-out validation set (same domain distribution).
- Zero-shot and few-shot performance on standard benchmarks (MMLU, HellaSwag, ARC).
- Monitor for memorization: check if training loss continues decreasing while validation loss increases.

---

## Problem 3: Debugging a Non-Converging Model

### Systematic Debugging Framework

**Step 1: Sanity Checks (5 minutes)**
- Verify expected loss at initialization: for C-class classification with cross-entropy, initial loss should be approximately -ln(1/C). For C=1000, that is ~6.9. If initial loss is much higher, something is wrong with the model output layer or loss computation.
- Can the model overfit a single batch of 8-16 examples? Train for 100 steps with a reasonable learning rate. Loss should approach zero. If not, there is a bug in the model, loss function, or data pipeline.

**Step 2: Data Pipeline Verification (10 minutes)**
- Visualize training samples: do inputs look correct? Are labels correct?
- Check for NaN/Inf in inputs after preprocessing.
- Verify normalization: are inputs normalized to a reasonable range (zero mean, unit variance)?
- Check class balance: extreme imbalance may require focal loss or oversampling.

**Step 3: Gradient Diagnostics (10 minutes)**
- Print gradient norms per layer after one forward-backward pass.
- All zeros: check for disconnected computation graph (detached tensors), dead ReLUs (all negative inputs), or frozen parameters.
- Exploding (>1000): reduce learning rate, add gradient clipping, check weight initialization.
- Vanishing (<1e-7 in early layers): add skip connections, switch to GELU/SiLU, check initialization (He init for ReLU).

**Step 4: Learning Rate Diagnosis (10 minutes)**
- LR range test: start at 1e-7, increase exponentially, plot loss vs LR. The optimal LR is typically 1-10x below where loss starts to diverge.
- Common failure mode: LR too high causes loss to oscillate without decreasing. Looks like a plateau but is actually instability.
- Try: reduce LR by 10x. If loss starts decreasing, original LR was too high.

**Step 5: Architecture and Normalization (15 minutes)**
- For deep networks (>10 layers): add residual connections if not present.
- Add LayerNorm (for Transformers) or BatchNorm (for CNNs) if not present.
- Check BatchNorm: is model in training mode during training and eval mode during evaluation? BatchNorm running statistics can cause silent failures.
- Check dropout: is it disabled during evaluation? Is the rate too high (>0.5)?

**Step 6: Numerical Stability (10 minutes)**
- Log-sum-exp trick: if computing softmax manually, use the numerically stable version.
- Mixed precision: if using fp16, check for overflow/underflow. Loss scaling may need adjustment. Use bf16 instead if hardware supports it.
- Check for NaN in loss: add assertions. NaN typically comes from log(0), division by zero, or inf - inf.

---

## Problem 4: Understanding and Implementing Attention Mechanisms

### Key Concepts

**1. Scaled Dot-Product Attention**
- Q (queries), K (keys), V (values) are linear projections of the input: Q = XW_Q, K = XW_K, V = XW_V.
- Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) * V.
- Why scale by sqrt(d_k)? Without scaling, the dot products grow in magnitude with d_k (variance of the dot product of two random vectors is d_k). Large dot products push softmax into regions with tiny gradients (saturated softmax), making learning very slow.

**2. Multi-Head Attention**
- Split d_model into h heads, each with d_k = d_model / h.
- Each head learns a different attention pattern (e.g., one head for syntax, another for semantics).
- Concatenate head outputs, apply final linear projection.
- Computation cost is the same as single-head attention with full dimensionality.

**3. Self-Attention vs Cross-Attention**
- Self-attention: Q, K, V all come from the same sequence. Used in encoder and decoder self-attention.
- Cross-attention: Q from one sequence (decoder), K and V from another (encoder output). Used in encoder-decoder models for tasks like translation.

**4. Causal (Masked) Attention**
- For autoregressive generation: token at position i can only attend to positions 0 through i.
- Implemented by adding a triangular mask (-inf) to attention scores before softmax.
- Ensures the model cannot "see the future" during training with teacher forcing.

---

## Problem 5: Batch Normalization vs Layer Normalization

### Key Concepts

**1. Batch Normalization**
- Normalizes across the batch dimension for each feature.
- For a mini-batch of size B: compute mean and variance across B samples for each feature channel.
- Learned scale (gamma) and shift (beta) parameters per feature.
- During training: use batch statistics. During inference: use running exponential moving average.
- Problem: behavior changes between train and eval mode. Small batch sizes give noisy estimates. Does not work well for sequential data (variable length).

**2. Layer Normalization**
- Normalizes across the feature dimension for each sample.
- For each sample: compute mean and variance across all features in that layer.
- No dependence on batch size. Same behavior at train and eval time.
- Standard choice for Transformers and RNNs.

**3. When to Use Which**
- CNNs with large batches: BatchNorm (well-studied, effective).
- Transformers: LayerNorm (batch-independent, works with variable sequence lengths).
- RNNs: LayerNorm (batch statistics are noisy across variable-length sequences).
- Small batch sizes (e.g., batch size 1 in reinforcement learning): LayerNorm or GroupNorm.
- Pre-norm vs post-norm in Transformers: pre-norm (normalize before attention/FFN) trains more stably; post-norm (original paper) may give slightly better final performance with careful tuning.

### Walkthrough: Why Normalization Helps
- Smooths the loss landscape: without normalization, the loss surface has many sharp valleys. With normalization, the landscape is smoother, allowing larger learning rates and faster convergence.
- Reduces sensitivity to initialization: normalized activations stay in a reasonable range regardless of initialization scale.
- Acts as a regularizer: batch statistics introduce noise (each sample's normalization depends on the other samples in the batch), providing a mild regularization effect.

### Common Interview Follow-Ups
- "What happens if you apply BatchNorm with a batch size of 1?" -- The mean and variance are computed from a single sample, making them meaningless. Use LayerNorm, InstanceNorm, or GroupNorm instead.
- "Why does BatchNorm interact poorly with dropout?" -- Dropout changes the scale of activations during training but not at inference. When combined with BatchNorm, the running statistics collected during training (with dropout) do not match inference behavior (without dropout). This can degrade performance.
- "How does normalization affect the effective learning rate?" -- Normalization decouples the scale of weights from the scale of activations. This means the effective learning rate for a normalized layer depends only on the direction of the weight update, not the magnitude of the weights. This is why normalized networks are less sensitive to the initial learning rate.
