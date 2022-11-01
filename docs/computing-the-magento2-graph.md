# Computing the Magento 2 Package Graph

## Adding the packages to the workspace

```bash
git clone https://github.com/magento/magento2 --depth=1
find magento2/app/code/ -maxdepth 3 -name \composer.json -exec cat {} + \
    | jq '{name: .name, path: (.autoload["psr-4"]|keys[0] | gsub("[\\\\]"; "/")) }' \
    | jq --slurp \
    | jq 'reduce .[] as $i ({}; .[$i.name] = 
        {
            "root": "magento2/app/code/\($i.path)",
            "sourceRoot": "magento2/app/code/\($i.path)",
            "projectType": "library",
            "targets": {
                "build": {
                    "executor": "@nrwl/workspace:run-commands",
                    "options": {
                        "commands": [
                            {
                                "command": "mkdir -p vendor/ && rsync magento2/app/code/\($i.path) vendor/\($i.name) --exclude Test -r"
                            }
                        ]
                    }
                }
            },
            "implicitDependencies": [
                "php-executor"
            ]
        }
    )' > output.json
```