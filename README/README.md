AKTAU-ESA — n8n workflows (Demo Package)

Этот репозиторий содержит экспортированные воркфлоу n8n команды AK-ES (модули поддержки безопасности спутниковых операций: Validator, Operator Handoff, Case-Finder/Similarity, Health_status_AI).

Доступы к приватной инфраструктуре (DB / vector store / LLM ключи) в репозиторий не включены по причинам безопасности и стоимости.  
Для проверки приложены реальные артефакты: примеры входов, JSON-выходы модулей, а также демонстрация в техдокументации.


 Быстрая проверка
1) Результаты работы модулей: откройте `demo_outputs/`  
    там лежат реальные JSON-выходы прогонов (включая retrieval outputs для rules/casecards).
2) примеры входных данных: откройте `samples/`  
    CDM, health_status, сообщения оператора, а также features для Case-Finder.
3) Архитектура пайплайнов: откройте `workflows/`  
    экспортированные JSON воркфлоу n8n (структура нод, связки модулей, логика обработки).
4) Доказательства выполнения (input → output): смотрите техдокументацию в `docs/`

 Содержимое репозитория
 `workflows/` — экспортированные n8n-воркфлоу (архитектура и логика).
 `samples/` — примеры входов для модулей (данные для демонстрации).
 `demo_outputs/` — реальные результаты прогонов (outputs), включая результаты retrieval из vector store (rules/casecards).
 `docs/` — техдокументация и/или скриншоты executions (если приложены).
 `constraints/` — список внешних зависимостей и ограничения по запуску.

 Модули и артефакты проверки

 1) Validator
Назначение: проверка полноты/согласованности входных CDM/данных, выставление флагов/замечаний по данным.  
- Workflow: `workflows/01_validator.json`
- Input samples: `samples/sample_cdm.json`
- Demo output: `demo_outputs/01_validator_output.json`

 2) Operator Handoff
Назначение: анализ переписки/уведомлений между операторами, фиксация статусов/передачи, формирование handoff-сводки.  
- Workflow: `workflows/02_handoff.json`
- Input samples: `samples/sample_operator_message.txt`
- Demo output: `demo_outputs/02_handoff_output.json`

3) Case-Finder / Similarity Analyst
Назначение: поиск исторически похожих случаев сближения/цепочек CDM и ранжирование кандидатов по схожести.

Core rules (MAJOR gate):
- exact match по source family: `catalog_name/каталог`, `provider/originator`, `covariance_method`, `ephemeris_class`
- match по `maneuverable` bucket
- match по `object_type_pair`
- желательно exact match по `dominant_miss_axis_ric` (если поле присутствует)
- raw `ephemeris_name` exact match **не обязателен** (сравнение ведётся по классу)

- Workflow: `workflows/03_case_finder.json`
- Input samples:
  - `samples/sample_casefinder_request.json`
  - `samples/sample_cdm_source_features.json`
- Demo outputs:
  - `demo_outputs/03_case_finder_final_full.json` (полный JSON-ответ AI-Agent)
  - `demo_outputs/03_case_finder_final_summary.json` или `.txt` (краткая версия)
  - `demo_outputs/03_case_finder_rules_retrieval_topk.json` (retrieval rules)
  - `demo_outputs/03_case_finder_casecards_retrieval_topk.json` (retrieval casecards)

4) Health_status_AI
Назначение:анализ health snapshot, выявление проблем/лимитных нарушений, присвоение уровня критичности и ограничений/рекомендаций по операциям.  
- Workflow: `workflows/04_health_status_ai.json`
- Input samples: `samples/sample_health_status.json`
- Demo output: `demo_outputs/04_health_output.json`

---

Почему проект может не запуститься:
Часть нод требует внешних зависимостей:
- База данных (Postgres/Neon), в т.ч. DIFF-таблицы и фичи
- Vector store / embeddings / документы для retrieval
- LLM provider ключи (если AI-Agent обращается к внешней модели)

Подробно: `constraints/CREDENTIALS_REQUIRED.md` и `constraints/LIMITATIONS.md`.


Импорт воркфлоу в n8n (при наличии своих credentials)
1) n8n → Workflows → Import from File
2) Импортируйте JSON из `workflows/`
3) Создайте свои credentials (DB/LLM) и привяжите их в соответствующих нодах