# Project API

The Project API facilitates the management of base data for a life cycle assessment (LCA) in a structured way that
fits the AEC industry.
The API module holds base objects such as `Project` and `ProjectMember`

## Data Structure

![Data Structure](/wiki/project-structure.png)

### Project

The `Project` class is the object that holds the project information.
It is a essential object as it connects, groups, members, project sources and reporting schemas arcoss the two API modules.

```python
class Project:
    id: str
    project_id: str | None
    name: str
    client: str | None
    domain: str | None
    address: str | None
    city: str | None
    country: str | None
    image_url: str | None
    groups: list[ProjectGroup]
    stages: list[ProjectStage]
    members: list[ProjectMember]
    meta_fields: dict
```

### Project Group

The `ProjectGroup` collects a series of project members under a common name.
A group can have a project member as lead, which becomes responsible for that group.

```python
class ProjectGroup:
    id: str
    name: str
    project: Project
    members: list[ProjectMember]
    lead: ProjectMember | None
```

### Project Member

The `ProjectMember` is an object that connects a user with a project. A project member is project specific,
which means that a user can have multiple project members connected to it.
User information, such as email and name is not stored on the project member object, but can be retrieved from the `User` object itself, through the `user_id` field

```python
class ProjectMember:
    id: str
    user_id: str
    project_groups: list[ProjectGroup]
    leader_of: list[ProjectGroup]
    project: Project
```

### Project Stage

The `ProjectStage` class is a link model to create many-to-many relationships between projects and life cycle stages.

```python
class ProjectStage:
    project: Project
    stage: LifeCycleStage
```

### Life Cycle Stage

The `LifeCycleStage` is an object to represent the different life cycle stages used in a LCA calculation.

```python
class LifeCycleStage:
    id: str
    name: str
    category: str
    phase: str
    projects: list[Project]
```