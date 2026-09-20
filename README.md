# itb-plugin-archimate

The OTB Gherkin dialect for **ArchiMate models** and their alignment with the
European Interoperability Reference Architecture (EIRA).

**Spec first: the dialect exists, the service does not.** This dialect was
written as the acceptance test for generation 2 of the language — it adds actor
kinds, a value type with its own path language, and a conformance handler, and
it needed no change to the core language to do so. Its HTTP contract is stated
in `itb-plugin.yaml` and `dialect/component.yml`; build a service to that
contract and the dialect runs unchanged. Until then a feature using it compiles
but cannot run.

## The language it adds

Two actor kinds, `archimate-repository` and `eira-validator`, and the value
type `archimate:Model` with ArchiMate model queries as its path language:

```gherkin
Background:
  Given Repo is an archimate-repository at "http://archimate-validator:8091"
  And EIRA is an eira-validator at "http://archimate-validator:8091"

Scenario: the solution model aligns with EIRA
  When Architect gets model "solution-architecture" from Repo as $model
  Then $model is an archimate model
  And $model at "elements.where(type = 'ApplicationService')" should not be empty
  And $model should conform to "https://.../eira/StructureDefinition/SolutionArchitecture"
```

Because the type declares its own path language, `at "…"` on an ArchiMate model
is an ArchiMate query, while `at "…"` on a FHIR resource stays FHIRPath. The
core language does not know about either.

## The contract a service must meet

The GITB validator contract every other validator here speaks, plus two
endpoints for models and queries:

| | |
|---|---|
| `GET /archimate/api/models/<name>` | the model, in Open Exchange Format XML or JSON |
| `POST /archimate/api/query` `{content, query}` | `{output:[{value}]}` — an ArchiMate model query |
| `POST /eira/api/validate` (GITB TAR) | `/report/context/0/value` = the error count |

## Where the files go

`dialect/` is canonical. `itb-cli/src/sync-dialects.mjs` copies it into every
app that compiles features — the authoring workbench and the ITB manager, each
under `public/components/archimate/`. Edit it there and the next sync
overwrites you.

Generation 2 of the language only; there is no `steps-v1.yml`.
