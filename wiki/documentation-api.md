# Documentation API

The Documentation API module facilitates the collection of data for a life cycle assessment (LCA) in a structured way that
fits the AEC industry.
The Documentation API implements a structure for acquiring data fitting the industry's standard classification
schemes.
It allows for request and review functionality through the `Task` implementation and furthermore implements version
control to track changes.

## Data Structure

![Data Structure](/wiki/documentation-structure.png)

### Reporting Schema

To create a data structure that can hold BIM7aa, CCS, etc. we are going use a tree like structure.
The `ReportingSchema` will be the top level model holding references to `SchemaCategories`, which function like grouping
elements with a name and description.

Below is the `SchemaElement` that will hold the actual data.
`SchemaElement`s have a reference to a `DataSource` if their data originates there.
`SchemaElement`s and `SchemaCategorie`s are immutable to allow for version control.
The GraphQL interface for `SchemaElement`s and `SchemaCategorie`s will look like a normal CRUD interface, but the actual
implementation should make the objects immutable and implement the version control system.

#### Schema Template

To be able to share reporting schemas between project we will implement a `SchemaTemplate` model that can hold a "
template" of reporting schema.
That way we can easily copy the original schema between projects.
It should also make it possible to make a change to reporting schema template and the change can be pulled to every
project that uses that reporting schema.

```python
class SchemaTemplate:
    id: str
    name: str
    original: ReportingSchema
    schemas: list[ReportingSchema]
```

#### Reporting Schema

When the reporting schema is a template, then it will not have a `project_id`.

```python
class ReportingSchema:
    id: str
    name: str
    project: Project | None
    template: SchemaTemplate
    repository: Repository
    tasks: list[Task]
    categories: list[SchemaCategory]
```

#### Schema Category

```python
class SchemaCategory:
    id: str
    path: str
    name: str
    description: str
    reporting_schema: ReportingSchema
    elements: list[SchemaElements]
    commits: list[Commit]
    tasks: list[Task]
```

#### Schema Element

```python
class SchemaElement:
    id: str
    name: str
    quantity: float
    unit: str
    description: str
    result: dict
    source: ProjectSource
    category: SchemaCategory
    tasks: Task
    commits: Commit
```

### Project Source

The Schema Elements should be able to pull data from external sources such as a CSV upload, Speckle, GlassHouse, etc.
To enable that we will create a `Source` model that will work as an interface to these external data sources.
The `Source` model needs specific logic to handle each type of data source and furthermore it should be possible to
extend the list of data source types as we go.

```python
class ProjectSource:
    id: str
    type: str
    data_id: str
    name: str
    project: Project
    meta_fields: dict
    elements: list[SchemaElement]
    interpretation: dict
    author: ProjectMember
    updated: datetime
    data: tuple[list[str], list[dict]]
```

### Version Control

To version control the Reporting Schema we propose a Git like system.
For the MVP the version control system will only support a single `main` branch, but in the further branching and
tagging can be added.

#### Repository

The `Repository` is the top level model of the Git like system.
The repository will contain references to all the commits and to the reporting schema that it version controls.

```python
class Repository:
    id: str
    reporting_schema: ReportingSchema
    commits: list[Commit]
    head: Commit
```

#### Commit

Each commit will store the full state of the reporting schema at a given point in time.
The elements field of the `Commit` model will contain a list of ids pointing to the valid schema element and schema
categories.

```python
class Commit:
    id: str
    added: datetime
    parent: Commit
    repository: Repository
    author: ProjectMemer
    schema_categories: list[SchemaCategory]
    schema_elements: list[SchemaElement]
    tasks: list[Task]
    tags: list[Tag]
```

#### Tag

A Tag can be added to a commit, so that users can easily find a certain point-in-time by the tag name
```python
class Tag:
    id: str
    author: ProjectMember
    added: datetime
    name: str
    commit: Commit
```

### Tasks

To implement a `Request and Review` type of system, the `Task` class and `Comment` class have been created.
The `Task` makes it possible to write tasks that should be done and assign them to specific project members or project groups.
A task should be connected to either a `SchemaElement` or `SchemaCategory`.
The task be assigneed a `TaskStatus` of either `Pending`, `Completed` or `Approved` to mark it current stage.

```python
class Task:
    id: str
    name: str
    description: str
    due_date: date
    owner: ProjectMember
    assignee: ProjectMember | ProjectGroup
    status: TaskStatus
    comments: list[Comment]
    item: SchemaElement | SchemaCategory
```

The `Comment` class is an implementation such that users can write comments to tasks and thereby communicate through the platform,
if something is unclear in the task description.

```python
class Comment:
    id: str
    owner: ProjectMember
    text: str
    added: date
    task: Task
```