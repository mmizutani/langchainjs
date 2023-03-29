---
id: by9km
title: LangChain OpenAPI Agent
file_version: 1.1.2
app_version: 1.5.2
---

# How to use OpenAPI Agent Toolkit

This example shows how to load and use an agent with a OpenAPI toolkit.

OpenAPI v3の形式で仕様定義された外部APIを呼び出して得られた出力をOpenAIのChatGPT APIにコンテキストとして送信するチャットエージェントを作成するには、以下の手順を踏みます。

1.  `OpenAIChat`<swm-token data-swm-token=":langchain/src/llms/openai-chat.ts:81:4:4:`export class OpenAIChat extends LLM implements OpenAIInput {`"/>クラスのインスタンスをお好みのパラメータを指定して作成。

2.  チャットエージェントの入出力方法を指定したりデータフィードを付け加えたりするための `OpenApiToolkit` クラスのインスタンスを作成。

3.  作成した `OpenAIChat`<swm-token data-swm-token=":langchain/src/llms/openai-chat.ts:81:4:4:`export class OpenAIChat extends LLM implements OpenAIInput {`"/>と `OpenApiToolkit`<swm-token data-swm-token=":langchain/src/agents/agent_toolkits/openapi/openapi.ts:30:4:4:`export class OpenApiToolkit extends RequestsToolkit {`"/>のインスタンスを基に、 `createOpenApiAgent`<swm-token data-swm-token=":langchain/src/agents/agent_toolkits/openapi/openapi.ts:48:4:4:`export function createOpenApiAgent(`"/>ファクトリメソッドを呼び出して OpenApiAgent を作成。

4.  OpenApiAgent に入力テキストを渡して、お好みの外部APIを呼び出した上で、その出力をもってChatGPT APIを呼び出し、最終的なチャット出力を得る。

```
import * as fs from "fs";
import * as yaml from "js-yaml";
import { JsonSpec, JsonObject } from "langchain/tools";
import { createOpenApiAgent, OpenApiToolkit } from "langchain/agents";
import { OpenAI } from "langchain";

export const run = async () => {
  let data: JsonObject;
  try {
    const yamlFile = fs.readFileSync("openai_openapi.yaml", "utf8");
    data = yaml.load(yamlFile) as JsonObject;
    if (!data) {
      throw new Error("Failed to load OpenAPI spec");
    }
  } catch (e) {
    console.error(e);
    return;
  }

  const headers = {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
  };
  const model = new OpenAI({ temperature: 0 });
  const toolkit = new OpenApiToolkit(new JsonSpec(data), model, headers);
  const executor = createOpenApiAgent(model, toolkit);

  const input = `Make a POST request to openai /completions. The prompt should be 'tell me a joke.'`;
  console.log(`Executing with input "${input}"...`);

  const result = await executor.call({ input });
  console.log(`Got output ${result.output}`);

  console.log(
    `Got intermediate steps ${JSON.stringify(
      result.intermediateSteps,
      null,
      2
    )}`
  );
};
```

呼び出し処理の流れは、Mermaid sequence diagramで表すと、以下の通り。

<br/>

<!--MERMAID {width:100}-->
```mermaid
sequenceDiagram
User -->> `OpenApiToolkit`: chat input
`OpenApiToolkit` -->> ExternalApiWithOpenAPISpec: request
ExternalApiWithOpenAPISpec -->> `OpenApiToolkit`: response
`OpenApiToolkit` -->> OpenAIChatGPTApi: original text & external API response
OpenAIChatGPTApi -->> \`Lang: chat response
`OpenApiToolkit` -->> User: chat response

```
<!--MCONTENT {content: "sequenceDiagram<br/>\nUser \\-\\-\\>> `OpenApiToolkit`<swm-token data-swm-token=\":langchain/src/agents/agent_toolkits/openapi/openapi.ts:30:4:4:`export class OpenApiToolkit extends RequestsToolkit {`\"/>: chat input<br/>\n`OpenApiToolkit`<swm-token data-swm-token=\":langchain/src/agents/agent_toolkits/openapi/openapi.ts:30:4:4:`export class OpenApiToolkit extends RequestsToolkit {`\"/> \\-\\-\\>> ExternalApiWithOpenAPISpec: request<br/>\nExternalApiWithOpenAPISpec \\-\\-\\>> `OpenApiToolkit`<swm-token data-swm-token=\":langchain/src/agents/agent_toolkits/openapi/openapi.ts:30:4:4:`export class OpenApiToolkit extends RequestsToolkit {`\"/>: response<br/>\n`OpenApiToolkit`<swm-token data-swm-token=\":langchain/src/agents/agent_toolkits/openapi/openapi.ts:30:4:4:`export class OpenApiToolkit extends RequestsToolkit {`\"/> \\-\\-\\>> OpenAIChatGPTApi: original text & external API response<br/>\nOpenAIChatGPTApi \\-\\-\\>> \\`Lang: chat response<br/>\n`OpenApiToolkit`<swm-token data-swm-token=\":langchain/src/agents/agent_toolkits/openapi/openapi.ts:30:4:4:`export class OpenApiToolkit extends RequestsToolkit {`\"/> \\-\\-\\>> User: chat response<br/>\n<br/>"} --->

<br/>

Mermaid C4 Diagram

<br/>

<!--MERMAID {width:100}-->
```mermaid
C4Context
title System Context diagram for Internet Banking System
Enterprise\_Boundary(b0, "BankBoundary0") {
Person(customerA, "Banking Customer A", \`Toolki"A customer of the bank, with personal bank accounts.")
Person(customerB, "Banking Customer B")
Person\_Ext(customerC, "Banking Customer C", "desc")
Person(customerD, "Banking Customer D", "A customer of the bank,
with personal bank accounts.")
System(SystemAA, "Internet Banking System", "Allows customers to view information about their bank accounts, and make payments.")
Enterprise\_Boundary(b1, "BankBoundary") {
SystemDb\_Ext(SystemE, "Mainframe Banking System", "Stores all of the core banking information about customers, accounts, transactions, etc.")
System\_Boundary(b2, "BankBoundary2") {
System(SystemA, "Banking System A")
System(SystemB, "Banking System B", "A system of the bank, with personal bank accounts. next line.")
}
System\_Ext(SystemC, "E-mail system", "The internal Microsoft Exchange e-mail system.")
SystemDb(SystemD, "Banking System D Database", "A system of the bank, with personal bank accounts.")
Boundary(b3, "BankBoundary3", "boundary") {
SystemQueue(SystemF, "Banking System F Queue", "A system of the bank.")
SystemQueue\_Ext(SystemG, "Banking System G Queue", "A system of the bank, with personal bank accounts.")
}
}
}
BiRel(customerA, SystemAA, "Uses")
BiRel(SystemAA, SystemE, "Uses")
Rel(SystemAA, SystemC, "Sends e-mails", "SMTP")
Rel(SystemC, customerA, "Sends e-mails to")
UpdateElementStyle(customerA, $fontColor="red", $bgColor="grey", $borderColor="red")
UpdateRelStyle(customerA, SystemAA, $textColor="blue", $lineColor="blue", $offsetX="5")
UpdateRelStyle(SystemAA, SystemE, $textColor="blue", $lineColor="blue", $offsetY="-10")
UpdateRelStyle(SystemAA, SystemC, $textColor="blue", $lineColor="blue", $offsetY="-40", $offsetX="-50")
UpdateRelStyle(SystemC, customerA, $textColor="red", $lineColor="red", $offsetX="-50", $offsetY="20")
UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```
<!--MCONTENT {content: "C4Context<br/>\ntitle System Context diagram for Internet Banking System<br/>\nEnterprise\\_Boundary(b0, \"BankBoundary0\") {<br/>\nPerson(customerA, \"Banking Customer A\", \\`Toolki\"A customer of the bank, with personal bank accounts.\")<br/>\nPerson(customerB, \"Banking Customer B\")<br/>\nPerson\\_Ext(customerC, \"Banking Customer C\", \"desc\")<br/>\nPerson(customerD, \"Banking Customer D\", \"A customer of the bank,<br/>\nwith personal bank accounts.\")<br/>\nSystem(SystemAA, \"Internet Banking System\", \"Allows customers to view information about their bank accounts, and make payments.\")<br/>\nEnterprise\\_Boundary(b1, \"BankBoundary\") {<br/>\nSystemDb\\_Ext(SystemE, \"Mainframe Banking System\", \"Stores all of the core banking information about customers, accounts, transactions, etc.\")<br/>\nSystem\\_Boundary(b2, \"BankBoundary2\") {<br/>\nSystem(SystemA, \"Banking System A\")<br/>\nSystem(SystemB, \"Banking System B\", \"A system of the bank, with personal bank accounts. next line.\")<br/>\n}<br/>\nSystem\\_Ext(SystemC, \"E-mail system\", \"The internal Microsoft Exchange e-mail system.\")<br/>\nSystemDb(SystemD, \"Banking System D Database\", \"A system of the bank, with personal bank accounts.\")<br/>\nBoundary(b3, \"BankBoundary3\", \"boundary\") {<br/>\nSystemQueue(SystemF, \"Banking System F Queue\", \"A system of the bank.\")<br/>\nSystemQueue\\_Ext(SystemG, \"Banking System G Queue\", \"A system of the bank, with personal bank accounts.\")<br/>\n}<br/>\n}<br/>\n}<br/>\nBiRel(customerA, SystemAA, \"Uses\")<br/>\nBiRel(SystemAA, SystemE, \"Uses\")<br/>\nRel(SystemAA, SystemC, \"Sends e-mails\", \"SMTP\")<br/>\nRel(SystemC, customerA, \"Sends e-mails to\")<br/>\nUpdateElementStyle(customerA, $fontColor=\"red\", $bgColor=\"grey\", $borderColor=\"red\")<br/>\nUpdateRelStyle(customerA, SystemAA, $textColor=\"blue\", $lineColor=\"blue\", $offsetX=\"5\")<br/>\nUpdateRelStyle(SystemAA, SystemE, $textColor=\"blue\", $lineColor=\"blue\", $offsetY=\"-10\")<br/>\nUpdateRelStyle(SystemAA, SystemC, $textColor=\"blue\", $lineColor=\"blue\", $offsetY=\"-40\", $offsetX=\"-50\")<br/>\nUpdateRelStyle(SystemC, customerA, $textColor=\"red\", $lineColor=\"red\", $offsetX=\"-50\", $offsetY=\"20\")<br/>\nUpdateLayoutConfig($c4ShapeInRow=\"3\", $c4BoundaryInRow=\"1\")"} --->

<br/>

This file was generated by Swimm. [Click here to view it in the app](/repos/Z2l0aHViJTNBJTNBbGFuZ2NoYWluanMlM0ElM0FtbWl6dXRhbmk=/docs/by9km).
