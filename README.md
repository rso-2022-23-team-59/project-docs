# General information about the project

## Development

### First start

To start working you need a suitable development structure where all repositories are in the same directory

such as:

rso/
    project-docs/
    product-catalog/
    store-catalog/
    ...


Then you can start all microservices with

```bash
docker compose up
```

### Useful commands:

Run only one service
```bash
docker compose up product-catalog
```

Build only one service
```bash
docker compose up product-catalog --build
```
