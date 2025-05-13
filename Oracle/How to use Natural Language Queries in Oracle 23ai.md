#oracle #ai
#### 1. **Set Up an AI Profile**

To enable Select AI, you need to create an AI profile that connects your database to an AI provider (e.g., OpenAI, Cohere, or Azure OpenAI). This is done using the `DBMS_CLOUD_AI` package:[Oracle Documentation+3Oracle Cloud Docs+3Oracle Documentation+3](https://docs.public.oneportal.content.oci.oraclecloud.com/en-us/iaas/autonomous-database/doc/use-select-ai-generate-sql-natural-language-prompts.html?utm_source=chatgpt.com)

```sql
BEGIN
  DBMS_CLOUD_AI.CREATE_PROFILE(
    profile_name     => 'my_ai_profile',
    provider         => 'openai',
    credential_name  => 'my_openai_cred',
    schema_list      => 'SH',
    model            => 'gpt-4'
  );
END;
/
```

This configuration allows the LLM to access your database schema metadata, enabling it to generate accurate SQL queries based on your natural language prompts. [Rittman Mead Consulting+1Oracle Cloud Docs+1](https://www.rittmanmead.com/blog/2023/11/autonomous-database-select-ai/?utm_source=chatgpt.com)

#### 2. **Execute Natural Language Queries**

Once your AI profile is set up, you can run natural language queries directly in SQL Developer Web or Oracle APEX. For example:[Oracle Documentation](https://docs.oracle.com/en/database/oracle/sql-developer-web/sdwad/run-natural-language.html?utm_source=chatgpt.com)

```sql
SELECT AI 'List the top 5 products by sales last year';
```

This command will:

- Translate your prompt into SQL.
- Execute the SQL query.
- Return the results.[Oracle Documentation+1Ateam Oracle+1](https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/sql-generation-ai-autonomous.html?utm_source=chatgpt.com)
    
You can also use modifiers like `SHOWSQL` to view the generated SQL or `NARRATE` for a textual summary:[Oracle Cloud Docs](https://docs.public.oneportal.content.oci.oraclecloud.com/en-us/iaas/autonomous-database/doc/use-select-ai-generate-sql-natural-language-prompts.html?utm_source=chatgpt.com)

```sql
SELECT AI SHOWSQL 'List the top 5 products by sales last year';
SELECT AI NARRATE 'List the top 5 products by sales last year';
```

---

### 💡 Additional Features

- **Chat Mode**: Engage in a conversational interface with your data, allowing for iterative queries and follow-up questions.
    
- **Multilingual Support**: Interact with your database using various languages, enhancing accessibility for global teams.
    
- **Voice Commands**: Use voice inputs to query your database, similar to virtual assistants.[Oracle](https://www.oracle.com/autonomous-database/select-ai/natural-language/?utm_source=chatgpt.com)
    

---

### ⚙️ Prerequisites

- Access to Oracle Autonomous Database.
    
- An account with an AI provider (e.g., OpenAI, Cohere) and corresponding API credentials.
    
- Necessary privileges to execute the `DBMS_CLOUD_AI` package.[elufasys.com](https://elufasys.com/oracle-autonomous-database-fluent-in-human-ai-revolutionizes-natural-language-queries/?utm_source=chatgpt.com)[Oracle Documentation+2Rittman Mead Consulting+2Oracle Cloud Docs+2](https://www.rittmanmead.com/blog/2023/11/autonomous-database-select-ai/?utm_source=chatgpt.com)[Oracle Documentation+3Oracle Documentation+3Rittman Mead Consulting+3](https://docs.oracle.com/en/database/oracle/sql-developer-web/sdwad/run-natural-language.html?utm_source=chatgpt.com)
