# LCAcollect Documentation

Here you can find an overview of the documentation for the LCAcollect implementation

![IT Architecture](/profile/it-architecture.png)

# Backend

The backend is written in [Python](https://www.python.org/) using the [FastAPI](https://fastapi.tiangolo.com/) framework.
To add GraphQL functionality we are using [Strawberry](https://strawberry.rocks/).
The backend application containerized and deployed and orchestrated with [Kubernetes](https://kubernetes.io/)

## Modules
- [Documentation API](/wiki/documentation-api.md)
- [Project API](/wiki/project-api.md)

# Frontend

The frontend is written in [Typescript](https://www.typescriptlang.org/) and [React](https://reactjs.org/) as a Singe Page Application (SPA).
The SPA communicates with the backend through GraphQL.
As a style library we are using [MUI](https://mui.com)
