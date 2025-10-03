# Mental Health Advice Guard: Technical Report

## Summary

This report documents the development of a production-ready guardrail system for detecting personalized mental health advice in user-assistant conversations. The system achieves **92.98% accuracy** and **98.54% AUC-ROC** on test data, with careful attention to minimizing false positives to avoid blocking legitimate educational conversations.

**Time Investment**: 7 hours total (6 hours build + 1 hours write-up)

**Note**: The notebook is available at [01_guard_notebook.ipynb](01_guard_notebook.ipynb).
- Normally I don't use Jupyter notebook and code structure can be significantly improved by using python scripts and modular code. For this experiment I used Jupyter notebook to use colab environment.
- In future this code can be restructured using UV project for better dependency handling, code organization and modularity.
- MLflow is used for experiment tracking and model registry but in simple form just to show the possibilites.

---

## 1. Problem Definition & Approach

### 1.1 Challenge Definition
- **Task**: Binary classification of text as `advice` vs `not_advice`
- **Key Distinction**: Personalized recommendations vs. general mental health education
- **Production Requirements**: High precision to avoid false positives, reasonable recall to catch actual advice

### 1.2 Design Philosophy
The system prioritizes **precision over recall** to avoid blocking legitimate educational conversations while maintaining sufficient sensitivity to detect personalized advice. This aligns with production requirements where false positives (blocking good content) are more problematic than occasional false negatives.

---

## 2. Data Strategy & Processing

### 2.1 Dataset Selection
**Primary Dataset**: `Amod/mental_health_counseling_conversations` from HuggingFace
- **Rationale**: Contains real counseling Q&A pairs from public mental health platforms
- **Format**: Context-Response pairs (user question + psychologist answer)
- **Size**: 2,454 training examples after processing
- **Language**: English
- **Quality**: Professional responses with advice-giving focus

### 2.2 Labeling Strategy
Implemented rule-based classification using linguistic patterns:

**Advice Indicators** (52 keywords/phrases):
- Direct recommendations: "you should", "you need to", "I recommend"
- Medical/treatment advice: "see a doctor", "medication", "therapy"
- Crisis intervention: "call 911", "emergency", "crisis hotline"
- Behavioral instructions: "stop doing", "start doing", "practice"

**General Information Indicators** (15 patterns):
- General statements: "many people", "research shows", "typically"
- Educational content: "what is", "symptoms include", "characterized by"
- Empathetic responses: "I understand", "that sounds", "you're not alone"

### 2.3 Data Processing Pipeline
1. **Text Cleaning**: Normalized whitespace, removed special characters
2. **Classification Logic**: 
   - Advice score based on keyword frequency and context
   - Information score based on educational patterns
   - Threshold-based classification with manual validation
3. **Quality Control**: Manual review of edge cases and ambiguous examples
4. **Data Splits**: 70% train, 15% validation, 15% test

### 2.4 Dataset Statistics
- **Total Examples**: 3,507
- **Training Set**: 2,454 (70%)
- **Validation Set**: 526 (15%) 
- **Test Set**: 527 (15%)
- **Class Distribution**: Approximately balanced (52% advice, 48% not_advice)

---

## 3. Model Architecture & Training

### 3.1 Model Selection
**Base Model**: `nlptown/bert-base-multilingual-uncased-sentiment` [https://huggingface.co/nlptown/bert-base-multilingual-uncased-sentiment](https://huggingface.co/nlptown/bert-base-multilingual-uncased-sentiment)
- **Rationale**: Strong performance on text classification, finetuned with six languages: English, Dutch, German, French, Spanish, and Italian.
- **Parameters**: 167M parameters
- **Architecture**: Transformer encoder with classification head

### 3.2 Model Architecture Details
```python
- BERT Transformer: 167M parameters
- Dropout Layer: 0.3 dropout rate
- Classification Head: Linear layer (768 → 2)
- Loss Function: CrossEntropyLoss with class weights [0.48, 0.52]
```

### 3.3 Training Configuration
- **Framework**: PyTorch Lightning for robust training pipeline
- **Optimizer**: AdamW with learning rate 2e-5
- **Scheduler**: Linear warmup (10% of steps) + linear decay
- **Batch Size**: 64
- **Max Sequence Length**: 256 tokens
- **Early Stopping**: Patience of 3 epochs on validation F1
- **Hardware**: GPU-accelerated training (CUDA)

### 3.4 Training Results
- **Training Time**: ~14 minutes (4 epochs)
- **Best Validation F1**: 0.9486 (epoch 3)
- **Early Stopping**: Triggered after epoch 3 due to no improvement
- **Final Model**: Best checkpoint from epoch 3

---

## 4. Model Performance & Evaluation

### 4.1 Test Set Performance
| Metric | Score |
|--------|-------|
| **Accuracy** | 92.98% |
| **AUC-ROC** | 98.54% |
| **Weighted Precision** | 92.99% |
| **Weighted Recall** | 92.98% |
| **Weighted F1-Score** | 92.98% |

### 4.2 Per-Class Performance
| Class | Precision | Recall | F1-Score |
|-------|-----------|--------|----------|
| **Not Advice** | 93.82% | 92.81% | 93.31% |
| **Advice** | 92.06% | 93.17% | 92.61% |

### 4.3 Performance Analysis
- **High AUC-ROC (98.54%)**: Excellent class separation capability
- **Balanced Performance**: Similar performance across both classes
- **Production-Ready**: High precision minimizes false positives
- **Robust Recall**: 92.8% recall on advice class catches most personalized advice

---

## 5. Guard Decision System

### 5.1 `guard_decide()` Function Implementation
The production interface returns structured decisions:

```python
{
    'label': 'advice' | 'not_advice',
    'confidence': float,  # 0.0 to 1.0
    'advice_probability': float,  # Probability of advice class
    'rationale': str,  # Human-readable explanation
    'action': 'allow' | 'flag' | 'block',  # Recommended action
    'response_template': str | None  # Suggested response for blocked content
}
```

### 5.2 Action Thresholds
- **Block** (confidence > 0.85): High-confidence advice detection
- **Flag** (0.65 < confidence ≤ 0.85): Medium-confidence, requires human review
- **Allow** (confidence ≤ 0.65): Low risk, allow through

### 5.3 Response Templates
- **Block Template**: "I understand you're looking for guidance, but I can't provide personalized mental health advice. Please consider speaking with a qualified mental health professional."
- **Flag Template**: "This response may contain personalized advice. Please review before sending."

---

## 6. Production Considerations

### 6.1 MLflow Integration
- **Experiment Tracking**: All training runs logged with hyperparameters and metrics
- **Model Registry**: Versioned model storage for deployment
- **Model Serving**: REST API endpoint for real-time inference
- **Monitoring**: Performance tracking and drift detection capabilities

### 6.2 Deployment Architecture
- **Model Format**: MLflow PyFunc for framework-agnostic serving
- **API Interface**: RESTful endpoint accepting text input
- **Response Time**: <100ms for typical inputs
- **Scalability**: Horizontally scalable with load balancing

### 6.3 Error Handling (limited due to time constraints)
- **Graceful Degradation**: Returns error status with safe defaults
- **Input Validation**: Text length limits and encoding checks
- **Logging**: Comprehensive error logging for debugging
- **Fallback**: Conservative "flag" action on processing errors

---

## 7. Limitations & Known Issues

### 7.1 Data Limitations
1. **English Only**: Dataset limited to English language conversations
2. **Domain Specificity**: Trained primarily on counseling conversations
3. **Cultural Bias**: May not generalize to different cultural contexts
4. **Temporal Bias**: Training data may not reflect current terminology

### 7.2 Model Limitations
1. **Context Window**: Limited to 256 tokens, may truncate long conversations
2. **Nuance Detection**: May struggle with subtle or implicit advice
3. **Sarcasm/Irony**: Limited ability to detect non-literal language
4. **Multi-turn Context**: Doesn't consider conversation history
5. **Language Limitation**: Limited to English language conversations and one test with Italian did not yield correct result.


### 7.4 Production Limitations
1. **Latency**: BERT inference may be slow for high-throughput applications
2. **Resource Usage**: Requires GPU for optimal performance
3. **Model Size**: 669MB model size may limit deployment options
4. **Update Frequency**: No mechanism for continuous learning from production data

---

## 8. Future Improvements & Recommendations
1. **Multilingual Support**: 
   - Train on multilingual and more aligned datasets
   - Validate performance across languages

2. **Enhanced Data Collection**:
   - Collect more diverse conversation types in different languages.
   - Add Reddit mental health discussions for negative examples

3. **Model Optimization**:
   - Experiment with DistilBERT for faster inference
   - Implement model quantization for reduced memory usage

4. **Evaluation Improvements**:
   - Implement human evaluation with mental health professionals
   - Add adversarial testing with edge cases

5. **Code Refactoring**:
   - Refactor code with python script for better maintainability and readability
   - Add unit tests
   - Add documentation

---

## 9. Conclusion

The Mental Health Advice Guard successfully addresses the challenge of detecting personalized mental health advice with strong performance metrics and production-ready architecture. The system demonstrates:

1. **Strong Performance**: 92.98% accuracy with excellent precision-recall balance
2. **Production Readiness**: Basic implimentation with error handling, monitoring, and deployment capabilities
3. **Thoughtful Design**: Careful consideration of false positive impacts
4. **Clear Limitations**: current constraints and improvement areas

---

## Sample Predictions

### Correctly Classified Examples

**Advice (Blocked)**:
- Input: "You should definitely see a therapist for your depression symptoms."
- Confidence: 99.9%
- Action: Block
- Rationale: Contains direct recommendation with personalized language

**Not Advice (Allowed)**:
- Input: "Depression is a common mental health condition that affects many people."
- Confidence: 99.6%
- Action: Allow
- Rationale: General educational information without personalization

---
