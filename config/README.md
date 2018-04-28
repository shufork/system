# config center


test 
open : http://127.0.0.1:18001/{service-id}/{profile}/{git-branch}

## mapping rules

- `/{application}/{profile}[/{label}]`
- `/{application}-{profile}.yml`
- `/{label}/{application}-{profile}.yml`
- `/{application}-{profile}.properties`
- `/{label}/{application}-{profile}.properties`


## see
- `org.springframework.cloud.config.server.environment.JGitEnvironmentRepository`
- `org.springframework.cloud.config.server.environment.MultipleJGitEnvironmentRepository`
