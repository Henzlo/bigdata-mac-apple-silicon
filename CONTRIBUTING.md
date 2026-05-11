# Contributing

Thank you for considering contributing to this project! 🙌

## How to contribute

1. **Fork** this repository
2. **Create a branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes** and test them locally with `docker-compose up -d`
4. **Commit**: `git commit -m "feat: describe your change"`
5. **Push**: `git push origin feature/your-feature-name`
6. Open a **Pull Request** against `main`

## What we welcome

- Bug fixes for services not starting on Apple Silicon
- New ARM64-native image alternatives
- Additional services (Kafka, Zeppelin interpreters, Presto, etc.)
- Documentation improvements
- Troubleshooting tips
- New usage examples for Hive, HBase, Spark, Zeppelin

## Testing your changes

Always verify your changes work on Mac Apple Silicon before submitting:

```bash
docker-compose down -v      # clean slate
docker-compose up -d        # start fresh
docker-compose ps           # all services should be Up
```

Then verify:
- Hive connects in DBeaver at `localhost:10000`
- Zeppelin opens at `http://localhost:9091`
- HDFS NameNode opens at `http://localhost:9870`

## Reporting issues

Please include:
- Your Mac chip (M1 / M2 / M3 / M4)
- macOS version
- Docker Desktop version (`docker --version`)
- Output of `docker-compose ps`
- Relevant logs (`docker-compose logs <service-name>`)
