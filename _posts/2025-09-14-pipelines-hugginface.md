---
layout: post
title: ""
description: ""
resume: ""
image: "/assets/images/"
readTime: ""
excerpt: IA intelligence articielle SLM LLM Hugging Face Transformers CPU GPU Python Java Modèle pré-entraîné
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Hugging Face ?](#hugging-face-)
3. [LLM vs SLM vs Pipelines Hugging Face](#llm-vs-slm-vs-pipelines-hugging-face)
4. [Les pipelines Hugging Face](#les-pipelines-hugging-face)
   1. [Choix du modèle](#choix-du-modèle)
   2. [Utilisation des pipelines Hugging Face](#utilisation-des-pipelines-hugging-face)
   3. [Utilisation de modèles SLM](#utilisation-de-modèles-slm)
      1. [Génération de texte](#génération-de-texte)
      2. [Traduction automatique de texte](#traduction-automatique-de-texte)
      3. [Résumé de texte](#résumé-de-texte)
   4. [Allons un peu plus loin](#allons-un-peu-plus-loin)
      1. [Exemple d'utilisation des Spaces](#exemple-dutilisation-des-spaces)
5. [Exemple d'intégration de Pipeline Hugging Face dans une application](#exemple-dintégration-de-pipeline-hugging-face-dans-une-application)
6. [Conclusion](#conclusion)
7. [Ressources supplémentaires](#ressources-supplémentaires)

## Introduction

L'Intelligence Artificielle (*IA*) est partout, et elle s'invite même dans nos outils de développement. GitHub Copilot, Amazon CodeWhisperer, Tabnine, et bien d'autres encore, sont des assistants de codage qui utilisent l'IA pour aider les développeurs à écrire du code plus rapidement et plus efficacement.
Ces outils sont basés sur des modèles de langage de grande taille appelés LLM (*Large Language Models*) qui ont été entraînés sur d'énormes quantités de données de code source. Ils peuvent comprendre le contexte du code que vous écrivez et suggérer des améliorations, des corrections, ou même générer du code complet à partir de simples
descriptions.

Dans cet article, nous allons explorer comment utiliser les modèles de langage de Hugging Face, plateforme populaire pour l'IA qui propose une vaste collection de modèles pré-entraînés, y compris des LLM, des SLM (*Small Language Models*), et des modèles spécialisés pour diverses tâches de traitement du langage naturel.
Nous allons voir comment installer les bibliothèques nécessaires, comment choisir un modèle adapté à vos besoins, et comment intégrer ces modèles dans vos projets de développement.

## Hugging Face ?

Hugging Face est une entreprise et une communauté open source créée en 2016, initialement pour un chatbot, avant de devenir un acteur majeur du domaine de l’IA.
Son objectif est de démocratiser l’intelligence artificielle en rendant les modèles de pointe accessibles à tous. Si je devais comparer Hugging Face, je dirai que c'est le GitHub de l’IA, car elle est une plateforme collaborative où chercheurs, ingénieurs et développeurs peuvent :

- Explorer le [Model Hub](https://huggingface.co/models), bibliothèque contenant des milliers de modèles pré-entraînés (LLM, SLM, etc) pour réaliser des tâches comme la traduction, le résumé, la modération, l'analyse d'images ou d'audio, et bien plus encore.
- Accéder au [Datasets Hub](https://huggingface.co/datasets), bibliothèque de données pour entraîner ou peaufiner vos propres modèles.
- Utiliser des bibliothèques OpenSource comme *transformers*, *datasets* et *tokenizers*.
- Créer votre propres applications d'IA avec [Spaces](https://huggingface.co/spaces), plateforme hébergée et gratuite pour les projets OpenSoure.
- Optimiser des modèles (inférence accélérée pour GPU, CPU, ONNX).

## LLM vs SLM vs Pipelines Hugging Face

- **LLM (Large Language Models)** : Ce sont des modèles de langage de grande taille, souvent composés de milliards de paramètres. Ils sont capables de comprendre et de générer du texte de manière très fluide et cohérente. Des exemples populaires incluent ChatGPT, LLaMA ou encore Mistral.  
  Ces modèles sont généralement utilisés pour des tâches complexes telles que la génération de texte, la traduction automatique, et la compréhension du langage naturel.
- **SLM (Small Language Models)** : Ce sont des modèles de langage plus petits, avec moins de paramètres. Ils sont souvent utilisés pour des tâches spécifiques où la taille du modèle est un facteur critique, comme par exemples les applications mobiles ou les systèmes embarqués.  
  Bien qu'ils soient moins puissants que les LLM, ils peuvent être très efficaces pour des tâches ciblées.
- **Pipelines Hugging Face** : Hugging Face propose une bibliothèque appelée `transformers` qui facilite l'utilisation des modèles de langage, qu'ils soient LLM ou SLM.
  Les pipelines sont des abstractions de haut niveau qui permettent d'exécuter des tâches courantes telles que la classification de texte, la génération de texte, la traduction, et bien plus encore, sans avoir à se soucier des détails techniques de l'implémentation du modèle. Ils simplifient grandement le processus d'intégration des modèles dans vos applications.

## Les pipelines Hugging Face

### Choix du modèle

Hugging Face propose une vaste collection de modèles pré-entraînés. Vous pouvez parcourir les modèles disponibles sur le site [Hugging Face Model Hub](https://huggingface.co/models). Vous pouvez filtrer les modèles par type de modèle, tâche, langue, taille, et bien plus encore.

### Utilisation des pipelines Hugging Face

Voici un exemple simple de comment utiliser un pipeline de classification de texte avec le modèle `distilbert-base-uncased`, SLM léger et efficace pour la classification de texte :

```python
from transformers import pipeline
# Créer un pipeline de classification de texte
classifier = pipeline('sentiment-analysis', model='distilbert-base-uncased')
# Analyser le sentiment d'un texte
result = classifier("I love using Hugging Face models!")
print(result)
```

Le résultat d'une pipeline de classification de texte est un dictionnaire ou une liste de dictionnaires, où chaque dictionnaire contiendra le sentiment (le label) - "positif" ou "négatif" - et la confiance du modèle dans sa prédiction (le score) - valeur entre 0 et 1, qui, plus elle est proche de 1, plus grande est la confiance dans la
prédiction.

Par exemple, pour le texte "*I love using Hugging Face models!*", vous pouvez obtenir ce genre de résultats :

```json
[
  {
    'label': 'POSITIVE',
    'score': 0.9998
  }
]
```

ou

```json
[
  {
    'label': 'NEGATIVE',
    'score': 0.9987
  }
]
```

Le label indique que le sentiment est positif, et le score représente la confiance du modèle dans cette prédiction. Si la valeur du label est 'NEGATIVE', cela signifie que le sentiment est négatif. Sinon, il est positif. Le score est une valeur entre 0 et 1, où une valeur plus proche de 1 indique une plus grande confiance dans la
prédiction.

Dans le premier exemple de résultat, le label est "*POSITIVE*" avec une confiance de 99.98%, donc le sentiment est positif.  
Dans le second, le label est "*NEGATIVE*" avec une confiance de 99.87%, le sentiment est donc négatif.

Ce code crée un pipeline de classification de texte et analyse le sentiment d'une phrase. Le résultat sera une liste de dictionnaires contenant le label (positif ou négatif) et la confiance associée.

### Utilisation de modèles SLM

#### Génération de texte

Voici un exemple d'utilisation d'un modèle SLM pour la génération de texte avec le modèle `gpt2` :

```python
from transformers import pipeline
# Créer un pipeline de génération de texte
text_generator = pipeline('text-generation', model='gpt2')
# Générer du texte à partir d'une invite
result = text_generator("Once upon a time", max_length=100, num_return_sequences=1)
print(result)
```

Ce code crée un pipeline de génération de texte et génère du texte à partir d'une invite donnée. Le résultat sera une liste de dictionnaires contenant le texte généré. Par exemple :

```json
[
  {
    "generated_text": "Once upon a time there was a young boy who lived in a small village near Rennes."
  }
]
```

Vous l'avez vu, le modèle utilisé est `gtp2`. Certes, il n'est pas tout récent, mais parfait pour expérimenter et il tourne sur CPU ! C'est un modèle OpenAI rendu public.

#### Traduction automatique de texte

```python
from transformers import pipeline
# Créer un pipeline de traduction automatique d'un texte en anglais en français
translator = pipeline("translation", model="Helsinki-NLP/opus-mt-en-fr")
# Traduire du texte
result = translator("Machine learning is fascinating!")
print(result[0]["translation_text"])
```

Le résultat en français sera alors : *"L'apprentissage automatique est fascinant !"*

Si vous souhaitez l'inverse, c'est-à-dire une traduction de français vers anglais, ce modèle existe : [Helsinki-NLP/opus-mt-fr-en](https://huggingface.co/Helsinki-NLP/opus-mt-fr-en).

#### Résumé de texte

```python
from transformers import pipeline
# Créer un pipeline de résumé de texte
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
# Résumer du texte
text = """
Hugging Face is an open-source platform that provides thousands of pretrained AI models.
It allows researchers and developers to share models, datasets, and applications,
making state-of-the-art AI more accessible to everyone worldwide.
"""
print(summarizer(text, max_length=40, min_length=10)[0]["summary_text"])
```

Le résumé sera alors : *"Hugging Face is an open-source platform that democratizes AI by sharing models and datasets."*

### Allons un peu plus loin

Hugging Face ne se résume pas aux pipelines, il permet également de :

- Ré-entraîner (*fine-tuner* dans le jargon de l'IA) un modèle avec vos propres données, permettant de l'adapter ainsi à votre contexte (forum de discussion, économique...), à une domaine (médical, juridique...) :

  ```python
  from datasets import load_dataset
  from transformers import AutoTokenizer, AutoModelForSequenceClassification, TrainingArguments, Trainer

  # Charge un jeu de données depuis le Datasets Hub
  dataset = load_dataset("imdb")

  # Choisit un modèle et un tokenizer
  model_name = "distilbert-base-uncased"
  tokenizer = AutoTokenizer.from_pretrained(model_name)

  # Prépare les données
  def tokenize(batch):
      return tokenizer(batch["text"], padding=True, truncation=True)

  tokenized_datasets = dataset.map(tokenize, batched=True)

  # Charge le modèle
  model = AutoModelForSequenceClassification.from_pretrained(model_name, num_labels=2)

  # Définit les paramètres d'entraînement
  training_args = TrainingArguments(
      output_dir="./results",
      evaluation_strategy="epoch",
      per_device_train_batch_size=8,
      num_train_epochs=1
  )

  # Crée l'entraîneur (Trainer)
  trainer = Trainer(
      model=model,
      args=training_args,
      train_dataset=tokenized_datasets["train"].shuffle(seed=42).select(range(2000)),
      eval_dataset=tokenized_datasets["test"].select(range(500))
  )

  # Lance l'entraînement
  trainer.train()
  ```

  Résultat ? Vous avez entrainé un modèle `distilbert` avec vos propres données et ceci en quelques lignes seulement ! Mieux, vous pouvez en faire profiter la communauté et le sauvegarder/publier sur le *Model Hub* (`trainer.push_to_hub("mon-modele")`).

- Explorer via *Model Card* les modèles : description, licence, exemples, configuration...
- Intégrer vos modèles dans le cloud grâce aux *Inference Endpoints* ou à *transformers*, et ceci afin d'exécuter ces modèles directement dans votre navigateur (sans backend !)
- Faire partie intégrante de la communauté OpenSource, c'est-à-dire de publier votre modèle, jeux de données ou "Spaces" interactifs.

#### Exemple d'utilisation des Spaces

Vous voulez lancer une application en quelques lignes de code ? directement depuis votre navigateur ? GO !

- Créez un compte Hugging Face
- Allez dans *Spaces*
- Créez un nouvel espace
- Choisissez votre bibliothèque Python qui vous permettra de transformer votre modèle en interface Web (choix entre [Gradio](https://www.gradio.app/) le choix le plus simple, Static pour héberger un site statique, et Docker pour une API Web par exemple)
- Créez votre répo ou déposez votre code python (comme celui qui suit par exemple)
- Attendez quelques secondes... Votre application est déployée, accessible publiquement, et cerise sur le gâteau, versionnée ! Comme un répo GitHub !  

J'ai fait l'exercice et voici le rendu de mon application : ![Web App démo](/assets/2025/09/24/webApp1.png)

J'entre un prompt avec pour but de générer du texte et voici le résultat 😍 ![Web App Démo prompt génération](/assets/2025/09/24/webApp2.png)

Maintenant, je tente de traduire le texte généré. ![Web App Démo prompt traduction](/assets/2025/09/24/webApp3.png).

Enfin, je demande de résumer le texte obtenu après traduction ![Web App Démo prompt résumé](/assets/2025/09/24/webApp4.png)

> **💡Tips**  
> Retrouvez mon [répo](https://huggingface.co/spaces/CedricSA/demo-article/tree/main) et mon [application de démo](https://huggingface.co/spaces/CedricSA/demo-article).

## Exemple d'intégration de Pipeline Hugging Face dans une application

TODO

## Conclusion

L'utilisation des modèles de Hugging Face et des pipelines facilite grandement l'intégration de l'IA dans vos projets de développement. Que vous utilisiez des LLM pour des tâches complexes ou des SLM pour des applications plus légères, Hugging Face offre une solution accessible et puissante pour exploiter la puissance de l'IA. N'hésitez pas
à explorer les nombreux modèles disponibles et à expérimenter avec différentes tâches pour voir comment l'IA peut améliorer vos flux de travail de développement.

## Ressources supplémentaires

- [Hugging Face Model Hub](https://huggingface.co/models)
- [Documentation Hugging Face Transformers](https://huggingface.co/docs/transformers/index)
- [Tutoriels Hugging Face](https://huggingface.co/course/chapter1)
