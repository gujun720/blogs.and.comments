---
title: "Passing the Turning Point of AI Transformation"
date: 2020-08-12T16:40:50+08:00
draft: false
tags: [技术,en]
categories: [正文]
canonicalUrl: https://towardsdatascience.com/passing-the-turning-point-of-ai-transformation-4855bc9742a1?source=your_stories_page----------------------------------------&gi=2a40fcde861a
---

When talking about the open-source AI projects, people would think of the model framework projects like Google TensorFlow, PyTorch, etc. Since the model framework is the critical component while training the AI models, those projects usually receive the most attention. But Artificial Intelligence (AI) is not a technology. AI is a complicated tech field involving several sub-areas and many different components.

## The turning point of AI transformation

Generally speaking, the turning point of technology upgrades is the point where the returns of technology upgrades go far beyond the cost. When it applies to AI transformation, it would involve fundamental factors containing models (algorithms), model inference, and data service.

While talking about the models, we need to ask ourselves about the expectation of leveraging AI technologies. If we want to use AI technologies to beat and replace human workers, for example, to replace all the customer support specialists with the AI-powered conversational bot. Then the demands of the AI models will be pretty high and will not even be achievable in the short term.

If we want to alleviate our customer support specialists from the tedious daily routines, which means we intend to leverage AI technologies as an amplifier to improve human productivity and capability, then the models today are already good enough in many scenarios.

It sounds inspiring. However, one heated debate regarding models is that, although some models are available to the public, the best models (the state-of-the-art models, SOTA) are not. The companies which could hire AI scientists with the own those SOTA models. So would I lose the competitive advantage if I only use the public models?

People are scratching their heads over this because they assume models with greater effectiveness would deliver higher business value. Yet this could be wrong. In most cases, the relationship between model effectiveness and business value is neither linear nor monotonically increasing. The graph of the function should look like below.

It is a piecewise function. In the first phase, before the model is practical in the application scenario, there is no business value. In the second phase, although a better model theoretically should have better performance (response time, effectiveness, etc.), it could be not that obvious in the real world scenario. Let us take a look at the below circumstance.

Before a doctor could confirm if a patient has lung infections, the doctor needs to take the CT images of the suspicious patient’s lungs. It would generate around 300 CT images. And an experienced doctor would have to spend 5–15 minutes on studying these hundreds of CT images. Normally, it would not be a problem if a doctor only deals with a small number of patients. However, in extreme cases such as the COVID-19, doctors are overwhelmed by the surge of patients.

The good news is data scientists have tried to help doctors by the computer vision technology. They trained the models which could process the hundreds of CT images and provide diagnostic suggestions in seconds. Hence doctors only need to take 1 minute to review the results generated by the models.

So before applying machine learning technology, it would take an average of 10 minutes to review the results generated in one CT scan, now it would take about 1 minute. The productivity improvement is almost 90%.

What if we have a faster model which would only need 3 seconds to generate the results? From 1 minute and 5 seconds to 1 minute and 3 seconds, it does not seem attractive.

What if we have a more effective model that could raise the accuracy from 80% to 90%? Could doctors review fewer results? The answer is no because, although the model could only be wrong in 1 of 10, we could not know which one is incorrect. Thus the final reviewer has to go through all the results. As a result, it would not save more diagnostic time.

Furthermore, to lower the cost of the model inference service, we sometimes would rather sacrifice the model effectiveness. Here’s a real example from our user, they are a business intelligence platform that holds 55 million images of trademarks. The company wanted to provide a service that allows users to search the owners of these trademarks. Users perform the search by uploading trademark images as the input query instead of giving the keywords.

The technology behind is computer vision, for instance, the VGG model. If the company runs the model inference on the back end server, they have to allocate and reserve the hardware resources in the data center. Another choice is to deploy a smaller model so that the company could put the model inference on edge devices (the smartphone in most cases). It will certainly cut off the cost of the expensive model inference hardware like GPU. It is another example that SOTA models are impossible to be competitive in all the scenarios.

We are already at the turning point of AI transformation. Then the question is how we could pass the turning point and adopt the AI technologies to empower our business.

A usable model is a prerequisite. However, we would not be able to develop an AI program at ease if we only have the model. Like in the traditional applications, data serving is always a critical piece. And we can see it is becoming an essential component in AI adoption nowadays. That is why we initiated the open-source project-Milvus to accelerate AI adoption.

## The data challenge of AI adoption

Most of the data we try to process through AI technologies is unstructured. We expect the Milvus project to provide a solid foundation for unstructured data service.

People usually categorize data into three types, structured data, semi-structured data, and unstructured data. Structured data includes numbers, dates, strings, etc. Semi-structured data usually comprises text information in certain formats, such as various computer systems logs. Unstructured data involves pictures, video, voice, natural language, and any other data that could not be directly processed by the computer.

It is estimated unstructured data accounts for at least 80% of the overall digital data universe. For example, you may send and receive several kilobytes text messages with your family, friends, or colleague every day. But even if you only take one photo on your mobile device, let’s say it’s iPhone 11 with a 12-megapixel camera, it would be several megabytes. And what if you take a video shot with 720p resolution?

People have developed technologies like relational databases, big data technologies to process structured data efficiently. And semi-structured data can be handled by the text-based search engine like Lucene, Solr, Elastic Search, etc. However, for unstructured data, the large chunk of all data, people didn’t have effective analytical methods in the past. Until the rise of Deep Learning technology in recent years, the development of unstructured data processing had been unthinkable.

## Unstructured data service

Embedding, a Deep Learning jargon, refers to transforming unstructured data into feature vectors through the models. Since a feature vector is a numeric array, it’s easy to be processed by computers. Thus the analysis of unstructured data could be translated to vector computation.

One most common argument is feature vector seems to be the intermediate result in the unstructured data processing. Is it necessary to build up a general-purpose vector similarity search engine? Should it be included in the models?

From my perspective, a feature vector is not just the intermediate result. It is the knowledge representation of unstructured data in Deep Learning scenarios. It’s also known as feature learning.

Another argument is, since a feature vector also contains numerical values, why not perform vector computation upon existing data processing platforms like databases or computing framework such as Spark.

To be precise, a vector consists of a list of numbers. It leads to two significant differences between vector computation and numerical operations.

First, the most frequent operations of vectors and numbers are different. For numbers, addition, subtraction, multiplication, and division are the most common operations. But for vectors, the most common requirement is to calculate the similarity. You see, here I am giving the formula for computing Euclidean distance, and the computation of vectors is much higher than the ordinary numerical calculation.

Secondly, the index organization of the data is different. Between two numbers, we can compare the values with each other. So we could create the number index based on the algorithm like B tree. But between two vectors, we could not perform the comparison. We can only calculate the similarity between them. So the vector index is usually based on algorithms like approximate nearest neighbor ANN algorithm.

Because of these significant differences, we find it hard to meet the requirements of vector analysis with the traditional database and big data technologies. The algorithms they support and the scenarios they focus on are all different.

We are not reinventing the wheels in the Milvus project.

## To find out the use cases

[https://github.com/milvus-io/bootcamp](https://github.com/milvus-io/bootcamp)
