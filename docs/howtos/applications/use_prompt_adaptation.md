# Automatic language adaptation


This is a tutorial notebook showcasing how to successfully use ragas with data from any given language. This is achieved using Ragas prompt adaptation feature. The tutorial specifically applies ragas metrics to a Hindi RAG evaluation dataset.

## Dataset
Here I’m using a dataset containing all the relevant columns in Hindi language. 

```{code-block} python

from datasets import load_dataset, Dataset

hindi_dataset = load_dataset("explodinggradients/amnesty_qa","hindi")
hindi_dataset
```

```{code-block}
DatasetDict({
    train: Dataset({
        features: ['question', 'ground_truths', 'answer', 'contexts'],
        num_rows: 20
    })
})
```

### Adapt metrics to target language

Import any metrics from Ragas as required and adapt and save each one of them to the target language.

```{code-block} python

from ragas.metrics import (
    faithfulness,
    answer_correctness,
)

target_language = "hindi"
faithfulness.adapt(target_language)
faithfulness.save()
answer_correctness.adapt(target_language)
answer_correctness.save()
```

The prompts belonging to respective metrics will be now automatically adapted to the target language. The save step saves it to `.cacha/ragas` by default to reuse later.  Next time when you do adapt with the same metrics, ragas first checks if the adapted prompt is already present in the cache. 

Let’s inspect the adapted prompt belonging to the answer correctness metric

```{code-block} python
print(answer_correctness.correctness_prompt.to_string())
```
```{code-block}
Extract the following from the given question and the ground truth

question: "सूरज को क्या चलाता है और इसका प्राथमिक कार्य क्या है?"
answer: "सूर्य परमाणु विघटन द्वारा संचालित होता है, जो पृथ्वी पर परमाणु रिएक्टरों के समान होते हैं, और इसका प्राथमिक कार्य सौरमंडल को प्रकाश प्रदान करना है।"
ground_truth: "सूर्य वास्तव में परमाणु संयोजन द्वारा चलाया जाता है, न कि विखंडन द्वारा। इसके केंद्र में, हाइड्रोजन परमाणु हीलियम बनाने के लिए मिल जाते हैं, जिससे बहुत अधिक ऊर्जा मुक्त होती है। यह ऊर्जा ही सूर्य को जलाती है और जीवन के लिए महत्वपूर्ण ताप और प्रकाश प्रदान करती है। सूर्य का प्रकाश भूमि की जलवायु प्रणाली में भी महत्वपूर्ण भूमिका निभाता है और मौसम और समुद्री धाराओं को चलाने में मदद करता है।"
Extracted statements: [{{"statements that are present in both the answer and the ground truth": ["सूर्य का प्राथमिक कार्य प्रकाश प्रदान करना है"], "statements present in the answer but not found in the ground truth": ["सूर्य पारमाणु विखंडन द्वारा संचालित होता है", "पृथ्वी पर पारमाणु रिएक्टरों के समान"], "relevant statements found in the ground truth but omitted in the answer": ["सूर्य पारमाणु संयोजन द्वारा संचालित होता है, न कि विखंडन", "इसके कोर में, हाइड्रोजन परमाणु हीलियम बनाने के लिए मिलकर तेजी से जलते हैं, जिससे बहुतायत ऊर्जा मुक्त होती है", "यह ऊर्जा जीवन के लिए आवश्यक गर्मी और प्रकाश प्रदान करती है", "सूर्य का प्रकाश पृथ्वी की जलवायु प्रणाली में महत्वपूर्ण भूमिका निभाता है", "सूर्य मौसम और समुद्री धाराओं को चलाने में मदद करता है"]}}]

question: "पानी का उबलने का बिंदु क्या है?"
answer: "पानी का उबलने का बिंदु समुद्री स्तर पर 100 डिग्री सेल्सियस है।"
ground_truth: "पानी का उबलने का बिंदु समुद्र तल पर 100 डिग्री सेल्सियस (212 डिग्री फारेनहाइट) होता है, लेकिन यह ऊचाई के साथ बदल सकता है।"
Extracted statements: [{{"statements that are present in both the answer and the ground truth": ["पानी का उबलने का बिंदु समुद्री स्तर पर 100 डिग्री सेल्सियस पर होता है"], "statements present in the answer but not found in the ground truth": [], "relevant statements found in the ground truth but omitted in the answer": ["ऊचाई के साथ उबलने का बिंदु बदल सकता है", "पानी का उबलने का बिंदु समुद्री स्तर पर 212 डिग्री फारेनहाइट होता है"]}}]

question: {question}
answer: {answer}
ground_truth: {ground_truth}
Extracted statements:
```

The instruction and key objects are kept unchanged intentionally to allow consuming and processing results in ragas.  During inspection, if any of the demonstrations seem faulty translated you can always correct it by going to the saved location. 

## Evaluate

```{code-block} python
from ragas import evaluate

ragas_score = evaluate(dataset['train'], metrics=[faithfulness,answer_correctness])
```

You will observe much better performance now with Hindi language as prompts are tailored to it.

