<template>
<div>
  <input type="file" @change="processFile" />
  <textarea v-model="questionsCSV" placeholder="Questions CSV"></textarea>
  <textarea v-model="answersCSV" placeholder="Answers CSV"></textarea>
</div>
</template>

<script setup lang="ts">
import { defineComponent, ref } from "vue";
import Papa from "papaparse";

const questionsCSV = ref("");
const answersCSV = ref("");

const processFile = (event: Event) => {
const file = (event.target as HTMLInputElement).files?.[0];
if (file) {
  const reader = new FileReader();
  reader.onload = (e) => {
    const content = e.target?.result;
    if (typeof content === "string") {
      const jsonData = convertXMLToJson(content);
      generateCSV(jsonData);
    }
  };
  reader.readAsText(file);
}
};

const convertXMLToJson = (xmlString: string) => {
const parser = new DOMParser();
const xmlDoc = parser.parseFromString(xmlString, "application/xml");

let questions = [];
let answers = [];

const questionNodes = xmlDoc.getElementsByTagName("QUESTION");
for (let i = 0; i < questionNodes.length; i++) {
  const question = questionNodes[i];
  const questionID = question.getAttribute("ID") || "";
  const description = question.getAttribute("DESCRIPTION") || "";
  const topic = question.getAttribute("TOPIC") || "";
  const status = question.getAttribute("STATUS") || "";

  questions.push({
    "Question ID": questionID,
    Description: description,
    Topic: topic,
    Status: status,
  });

  const choices = question.getElementsByTagName("CHOICE");
  for (let j = 0; j < choices.length; j++) {
    const choice = choices[j];
    const choiceID = choice.getAttribute("ID") || "";
    const contentNode = choice.getElementsByTagName("CONTENT")[0];
    const content = contentNode.textContent?.trim() || "";

    answers.push({
      "Answer ID": `${questionID}-${choiceID}`,
      "Question ID": questionID,
      Content: content,
    });
  }
}

return { questions, answers };
};

const generateCSV = (jsonData: { questions: any[]; answers: any[] }) => {
questionsCSV.value = Papa.unparse(jsonData.questions);
answersCSV.value = Papa.unparse(jsonData.answers);
};
</script>
