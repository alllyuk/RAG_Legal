# RAG_Legal
Документация по RAG-системе для работы с нормативно-правовыми актами (НПА) Ханты-Мансийского автономного округа.

Разработано решение, которое строит базу знаний на основе предоставленных данных (например, НПА). На основе извлеченной из базы знаний информации и запроса в LLM система хорошо отвечает на поставленные вопросы, опираясь на контекст из базы знаний. Это позволяет возвращать результат куда более лучший, чем при генерации ответа с помощью LLM без поиска контекста.

В ноутбуке `master.ipynb` приведены параграфы с написанным проектом и примером его запуска для работы с нормативно-правовыми актами. 

## Инструкция по запуску
1. Клонирование репозитория
```
git clone https://github.com/alllyuk/RAG_Legal
```
2. Установка зависимостей
```
pip install -r requirements.txt
```
3. Загрузка данных\
Для работы с базой знаний требуется указать название файла с базой знаний при инициализации класса `CustomRAGPipeline` в параметр `documents_path`.

4. Запуск кода из ноутбука\
Далее можно при необходимости настроить параметры запуска системы в словаре `config` (как настраивать, написано ниже).
Затем необходимо запускать все ячейки в данном ноутбуке, чтобы получить ответ и в конце сохранить датасет для оценки в ragas.

5. Работа через веб-интерфейс или с API\
Ниже в разделе **Запуск API и веб-интерфейса** описана инструкция по соответствующему варианту запуска кода.


### Настройка параметров запуска
С помощью настройки словаря `config` можно менять параметры запуска системы, например использовать другие LLM или менять ретривер для поиска документов.
```
config = { # The first value in every config-line is baseline version
    'model_name': 'llama3.1-8b-q4',  # llama3.1-8b-q4 / gemma-2-9b-it-simpo-q4 / tlite-q4
    'embed_model_name_short': 'e5l', # e5l (multilingual-e5-large) / bgem3 (bge-m3)
    'chunk_size': 512, # 512, 1024, 256
    'chunk_overlap': 128 # 128, 256, 64
    'llm_framework': 'VLLM', # VLLM / Ollama (Ollama is only for gemma2 model)
    'vectorstore_name': 'MILVUS', # MILVUS / FAISS
    'retriever_name': 'vectorstore', # 'BM25' / 'vectorstore' / 'ensemble' (ensemble is HybridSearch with BM25 and vectorstore)
    'retriever_k': 4, # how much chunks will return retiever: 4, if reranker, then retriever_k=30 (reranker returns 4)
    'compressor_name': None, # if needed post-processing of retiever results: None / 'cross_encoder_reranker' / 'gluing_chunks' 
    'chain_type': 'stuff', # base version of chain_type: 'stuff'
}

# If chosen a HybridSearch
ensemble_config = {
    'ensemble_retrievers_names': ['BM25', 'vectorstore'],
    'ensemble_retrievers_weights': [0.4, 0.6], 
}

llama_config = {
    'repo_id': 'lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF',
    'filename': 'Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf',
    'tokenizer': 'hugging-quants/Meta-Llama-3.1-8B-Instruct-AWQ-INT4'
    }

gemma_config = {
    'repo_id': "mannix/gemma2-9b-simpo",
    'llm_framework': 'Ollama'
    }

tlite_config = {
    'repo_id': 'mradermacher/saiga_tlite_8b-GGUF',
    'filename': 'saiga_tlite_8b.Q4_K_M.gguf',
    'tokenizer': 'IlyaGusev/saiga_tlite_8b'
    }

reranker_config = {
    'reranker_model': "BAAI/bge-reranker-v2-m3",
    'retriever_k': 30
}

```
### Запуск API и веб-интерфейса
Реализован рабочий API. Запускается командой
```
uvicorn api:app --reload --port [api_port]
```
Веб-интерфейс поднимается командой
```
python webui.py --api_port [api_port] --demo_port [demo_port]
```
На localhost:[demo_port] поднимается локальная демо-версия чат-бота

![image](https://github.com/user-attachments/assets/bb981fd1-836a-44aa-ba47-7857f7f4c03b)

Request {"question":  "string"}
Response {"response": "string", "context": "string"}

### Конфигурация параметров, которая использовалась при оптимальном результате
Этот запуск был осуществлен в ноутбуке [**master_final.ipynb**](master_final.ipynb)
```
config = {
    'model_name': 'gemma-2-9b-it-simpo-q4',  # llama3.1-8b-q4 / gemma-2-9b-it-simpo-q4 / tlite-q4
    'embed_model_name_short': 'e5l', # e5l (multilingual-e5-large) / bgem3 (bge-m3) / ubgem3 (USER-bge-m3)
    'chunk_size': 512, # либо 512/128, 1024/256, 256/64 (или ?2048/256?)
    'chunk_overlap': 128,
    'llm_framework': 'VLLM', # VLLM, LLamaCpp, Ollama
    'vectorstore_name': 'MILVUS', # база данных MILVUS / FAISS
    'retriever_name': 'ensemble', # 'vectorstore' / 'ensemble' (BM25 + vertorstore)
    'retriever_k': 4,
    'compressor_name': 'gluing_chunks', # None / 'cross_encoder_reranker' / 'gluing_chunks' 
    'chain_type': 'stuff',
}
```
