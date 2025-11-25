---
layout: post
title: "Hugging Face, LA plateforme qui démocratise l’intelligence artificielle !"
description: "Dans cet article, nous allons découvrir comment exploiter les modèles de langage de Hugging Face, comment choisir des modèles adaptés à vos besoins, et comment intégrer ces derniers dans vos projets de développement."
resume: "Dans cet article, nous allons découvrir comment exploiter les modèles de langage de Hugging Face, comment choisir des modèles adaptés à vos besoins, et comment intégrer ces derniers dans vos projets de développement."
image: "/assets/images/hugging_face.svg"
readTime: "15"
excerpt: IA intelligence artificielle SLM LLM Hugging Face Transformers CPU GPU Python Java Modèle pré-entraîné fine-tuning
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Hugging Face ?](#hugging-face-)
3. [LLM, SLM et Pipelines Hugging Face](#llm-slm-et-pipelines-hugging-face)
4. [Les pipelines Hugging Face](#les-pipelines-hugging-face)
   1. [Choix du modèle](#choix-du-modèle)
   2. [Utilisation des pipelines Hugging Face](#utilisation-des-pipelines-hugging-face)
   3. [Utilisation de modèles légers (SLM)](#utilisation-de-modèles-légers-slm)
      1. [Génération de texte](#génération-de-texte)
      2. [Traduction automatique de texte](#traduction-automatique-de-texte)
      3. [Résumé de texte](#résumé-de-texte)
   4. [Allons un peu plus loin](#allons-un-peu-plus-loin)
      1. [Fine-tuning](#fine-tuning)
      2. [Utilisation des Spaces](#utilisation-des-spaces)
      3. [Licences, sécurité et bonnes pratiques](#licences-sécurité-et-bonnes-pratiques)
5. [Exemple d'intégration de Pipeline Hugging Face dans une application Python](#exemple-dintégration-de-pipeline-hugging-face-dans-une-application-python)
6. [Conclusion](#conclusion)
7. [Ressources supplémentaires](#ressources-supplémentaires)
8. [❓ FAQ Hugging Face](#-faq-hugging-face)

## Introduction

L'Intelligence Artificielle (*IA*) est partout, et elle s'invite même dans nos outils de développement. *GitHub Copilot*, *Amazon CodeWhisperer*, *Tabnine*, et bien d'autres encore, sont des assistants de codage qui utilisent l'IA pour nous aider, développeurs, à écrire du code plus rapidement et plus efficacement.

Ces outils sont basés sur des modèles de langage de grande taille appelés LLM (*Large Language Models*), entraînés sur d'énormes quantités de données de code et de texte. Ils comprennent le contexte de ce que vous écrivez et peuvent suggérer des améliorations, des corrections, ou même générer du code complet à partir d'une simple description.

Dans cet article, nous verrons comment utiliser **Hugging Face** pour découvrir des modèles IA, les tester, les intégrer via les pipelines, et même les déployer facilement sur les *Spaces*.

## Hugging Face ?

Hugging Face est une entreprise et une communauté open source créée en 2016 par Clément Delangue, Julien Chaumond et Thomas Wolf. Initialement centrée sur un chatbot, elle est rapidement devenue un acteur majeur de l'IA Open Source.

Son objectif est de **démocratiser l’intelligence artificielle en rendant les modèles accessibles à tous**. Si je devais comparer Hugging Face, je dirais que c'est le GitHub de l’IA, car c'est une plateforme collaborative où chercheurs, ingénieurs et développeurs peuvent :

- Explorer le [Model Hub](https://huggingface.co/models), une immense bibliothèque de modèles pré-entraînés (LLM, SLM, etc.) pour la traduction, le résumé, la modération, l’analyse d’images ou d’audio, et bien plus encore.
- Accéder au [Datasets Hub](https://huggingface.co/datasets), un catalogue de jeux de données, prêts à l'emploi pour entraîner ou peaufiner vos modèles.
- Utiliser des bibliothèques OpenSource comme *transformers*, *datasets* et *tokenizers*.
- Créer facilement votre propre application d'IA avec [Spaces](https://huggingface.co/spaces), plateforme hébergée et gratuite pour les projets OpenSource.
- Optimiser l'inférence des modèles (GPU, CPU, ONNX...).

## LLM, SLM et Pipelines Hugging Face

- **LLM (Large Language Models)** : modèles de grande taille (souvent composés de milliards de paramètres) capables de comprendre et de générer du texte de manière très fluide et cohérente. Des exemples populaires incluent ChatGPT, LLaMA ou encore Mistral.  
  Ces modèles sont généralement utilisés pour des tâches complexes telles que la génération et la traduction de texte, la compréhension du langage naturel...

- **SLM (Small Language Models)** : dans cet article, j’utilise le terme *SLM* pour désigner des modèles plus petits que les LLM classiques (généralement quelques centaines de millions de paramètres), optimisés pour tourner sur CPU, appareils mobile ou systèmes embarqués.  
  Bien qu'ils soient moins puissants que les LLM, ils peuvent être très efficaces pour des tâches ciblées et de l'expérimentation.

- **Pipelines Hugging Face** : Les pipelines Hugging Face sont des outils prêts à l’emploi qui permettent d’utiliser un modèle en une seule ligne de code.  
  Ils regroupent automatiquement tout ce qu’il faut : le modèle, son tokenizer, et les étapes de préparation et de traitement des données.  
  Vous pouvez donc exécuter une tâche comme la classification, la génération de texte ou la traduction sans gérer manuellement les détails internes du modèle.  
  C’est la manière la plus rapide d’utiliser un modèle sans manipuler directement les composants bas niveau de `transformers`.

## Les pipelines Hugging Face

### Choix du modèle

Hugging Face propose une vaste collection de modèles pré-entraînés. Vous pouvez les parcourir sur le [Model Hub](https://huggingface.co/models), et les filtrer par type, tâche, langage, taille, licence, etc.

### Utilisation des pipelines Hugging Face

Voici un exemple simple avec un pipeline de classification de texte, utilisant un modèle `DistilBERT` fine-tuned pour le sentiment :

```python
from transformers import pipeline
# Créer un pipeline de classification de texte
classifier = pipeline('sentiment-analysis', model='distilbert-base-uncased-finetuned-sst-2-english')
# Analyser le sentiment d'un texte
result = classifier("I love using Hugging Face models!")
print(result)
```

Le résultat est une liste de dictionnaires, chacun contenant le sentiment (le label) - "positif" ou "négatif" - et la confiance du modèle dans sa prédiction (le score) - valeur entre 0 et 1, qui, plus elle est proche de 1, plus grande est la confiance dans la prédiction.

Par exemple, pour le texte "*I love using Hugging Face models!*", vous pouvez obtenir ce genre de résultats :

```json
[
  {
    'label': 'POSITIVE',
    'score': 0.9998
  }
]
```

### Utilisation de modèles légers (SLM)

#### Génération de texte

Voici un exemple d'utilisation d'un modèle léger pour la génération de texte avec le modèle `gpt2` tournant facilement sur CPU :

```python
from transformers import pipeline
# Créer un pipeline de génération de texte
text_generator = pipeline('text-generation', model='gpt2')
# Générer du texte à partir d'une accroche
result = text_generator("Once upon a time", max_length=100, num_return_sequences=1)
print(result)
```

Exemple de texte généré :

```json
[
  {
    "generated_text": "Once upon a time there was a young boy who lived in a small village near Rennes."
  }
]
```

`gpt2` n’est pas un modèle récent, mais il est idéal pour expérimenter.

#### Traduction automatique de texte

```python
from transformers import pipeline
# Créer un pipeline de traduction automatique d'un texte en anglais en français
translator = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
# Traduire du texte
result = translator("Machine learning is fascinating!")
print(result[0]["translation_text"])
```

Le résultat en français sera alors : *"L'apprentissage automatique est fascinant !"*

> **💡Tips**  
> Pour traduire de **FR**ançais en **EN**glish : [Helsinki-NLP/opus-mt-fr-en](https://huggingface.co/Helsinki-NLP/opus-mt-fr-en)

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

#### Fine-tuning

Hugging Face ne se résume pas aux pipelines, il permet également de réentraîner (*fine-tuner* dans le jargon de l'IA) un modèle sur vos propres données pour l'adapter à votre contexte ou domaine métier.

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

Résultat ? En quelques lignes, vous obtenez un modèle `distilbert` fine-tuné sur vos propres données  ! Mieux, vous pouvez en faire profiter la communauté et le publier sur le *Model Hub* : `trainer.push_to_hub("mon-modele")`.

#### Utilisation des Spaces

Hugging Face vous permet également d'explorer via *Model Card* les modèles (description, licence, exemples, configuration...), d'héberger vos modèles dans le cloud grâce aux *Inference Endpoints* ou via *transformers*, et ceci afin d'exécuter ces modèles directement dans votre navigateur (sans backend !). Enfin, il vous permet de faire partie intégrante de la communauté OpenSource, c'est-à-dire de publier votre modèle, jeux de données ou "Spaces" interactifs.

Vous voulez lancer une application en quelques lignes de code ? directement depuis votre navigateur ? J'ai fait l'exercice et voici comment faire !

- Créez un compte Hugging Face
- Allez dans *Spaces*
- Créez un nouvel espace
- Choisissez votre bibliothèque Python qui vous permettra de transformer votre modèle en interface Web (choix entre [Gradio](https://www.gradio.app/) le choix le plus simple, Static pour héberger un site statique, et Docker pour une API Web par exemple). Ici, j'ai choisi Gradio.
- Créez votre répo ou déposez votre code python (comme celui qui suit par exemple)
- Attendez quelques dizaines de secondes...
- Votre application est déployée, accessible publiquement, et cerise sur le gâteau, versionnée ! Comme un répo GitHub !  

Voici le rendu de mon application qui permet de générer du texte, de le traduire et/ou de le résumer.

Une fois les étapes sus-citées réalisées, mon application ressemble à ça :

![Web App démo](/assets/2025/09/24/webApp1.png)

J'entre un prompt avec pour but de générer du texte et voici le résultat 😍

![Web App Démo prompt génération](/assets/2025/09/24/webApp2.png)

Maintenant, je demande la traduction du texte généré :

![Web App Démo prompt traduction](/assets/2025/09/24/webApp3.png).

Enfin, je demande de résumer le texte obtenu après traduction ![Web App Démo prompt résumé](/assets/2025/09/24/webApp4.png)

> **💡Tips**  
> Retrouvez mon [répo](https://huggingface.co/spaces/CedricSA/demo-article/tree/main) et mon [application de démo](https://huggingface.co/spaces/CedricSA/demo-article).  
> Les Spaces sur le hardware cpu-basic (free) sont automatiquement mis en pause après 48h d’inactivité (afin d'économiser les ressources) ; les Spaces sur hardware « upgrade » ont des règles différentes et peuvent ne pas être mis en pause.
> Mais vous pouvez le redemarrer si vous souhaitez le tester 😉

#### Licences, sécurité et bonnes pratiques

Avant d'utiliser un modèles, pensez à :

- **Vérifier la licence** : chaque modèle sur le Hub a une licence (Apache-2.0, MIT, CC-BY...) et certaines interdisent l'usage commercial.
- **Vérifier la sécurité du modèle** : éviter d'exécuter un modèle ne production sans "audit", en effet, certains peuvent contenir du code Python exécuté à l'import.Pensez à vérifier les fichiers `model.py`, `configuration.py` entre autres.
- **Protéger les données sensibles (RGPD)** : anonymisez les données personnelles avant d’entraîner un modèle. Le fine-tuning peut mémoriser des informations privées.

## Exemple d'intégration de Pipeline Hugging Face dans une application Python

TODO

## Conclusion

Hugging Face s’est imposé comme un acteur central de l’IA moderne, non seulement grâce à son catalogue impressionnant de modèles, mais aussi grâce à ses outils simples, efficaces et accessibles. Que vous souhaitiez utiliser un modèle existant, fine-tuner un LLM ou un SLM, ou déployer une démo sur un Space, tout est pensé pour que vous puissiez passer de l’idée à la mise en production rapidement, même sans infrastructure lourde.

En maîtrisant les éléments présentés dans cet article — modèles, pipelines, fine-tuning et Spaces — vous disposez déjà de 80 % de ce qu’il faut pour intégrer l’IA dans vos projets, prototypes ou produits.

Et ce n’est qu’un début : la plateforme évolue sans cesse, les modèles deviennent plus légers, plus rapides et plus performants, et la communauté ne cesse de croître.

Bref : si vous voulez explorer, construire ou déployer de l’IA aujourd’hui, Hugging Face est probablement l’endroit le plus accueillant où commencer.

## Ressources supplémentaires

- [Model Hub](https://huggingface.co/models) : milliers de modèles pré-entraînés
- [DatasetHub](https://huggingface.co/datasets) : catalogue de jeux de données prêts à l’emploi
- [Documentation Transformers](https://huggingface.co/docs/transformers/index)
- [Documentation Pipelines](https://huggingface.co/docs/transformers/main_classes/pipelines)
- [Tutoriels officiels Hugging Face](https://huggingface.co/course)
- [Examples de Spaces](https://huggingface.co/spaces)
- [API Inference Endpoints](https://huggingface.co/inference-api)

## ❓ FAQ Hugging Face

**Hugging Face est-il gratuit ?**

**Oui**, la majorité des modèles, datasets, Spaces CPU, et l’utilisation de la librairie transformers sont gratuits. Certaines fonctionnalités avancées (GPU sur Spaces, Inference Endpoints managés) sont payantes.

**Puis-je utiliser un modèle sans GPU ?**

**Oui**. Beaucoup de modèles légers (SLM), quantifiés ou distillés fonctionnent très bien sur CPU. Hugging Face propose même des modèles spécialement optimisés pour cette contrainte.

**Quelle est la différence entre un LLM et un SLM ?**

Un LLM est un modèle très volumineux (des dizaines de milliards de paramètres).  
Un SLM est un modèle plus compact, conçu pour tourner sur CPU ou appareils low-resource.  
Les deux peuvent être utilisés via Hugging Face.

**Les pipelines sont-ils adaptés pour la production ?**

Dans certains cas, **oui** ! Mais ils restent parfaits pour prototyper et tester.Pour une application robuste ou performante, il est conseillé d'utiliser les classes bas niveau (AutoModel, AutoTokenizer) ou un Inference Endpoint.

**Puis-je fine-tuner un modèle gratuitement ?**

**Oui**, sur CPU ou GPU gratuit via les Spaces (avec des limitations). Sinon, localement.

**Quels sont les modèles recommandés pour débuter ?**

- **DistilBERT** pour la classification
- **GPT-2** pour la génération
- **BART** pour le résumé
- **Whisper** pour l’audio

Tous disponibles sur Hugging Face 😁
