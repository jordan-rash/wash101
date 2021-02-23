# wash 101

Demo resources for the `wash 101` demo. This demo walks through all of the subcommands currently contained in the `0.2.0` version of [wash](https://github.com/wasmcloud/wash). With the prerequisites below, you can run the commands to walk through each subcommand in `wash` for a full [wasmcloud](https://github.com/wasmcloud/wasmcloud) application.

## Prerequisites
- Cloned version of this repository
- `docker-compose`
- `docker`
- `wash`

## Commands used
```shell
## Building and signing table tennis actor
cd ./actor-table-tennis
wash claims sign --help
wash claims sign ./target/wasm32-unknown-unknown/release/table_tennis.wasm --name "Table Tennis" --http_server
wash claims inspect ./target/wasm32-unknown-unknown/release/table_tennis_s.wasm
wash keys gen --help
TABLE_TENNIS_SUBJECT=$(wash keys gen module -o json | jq '.seed' | tr -d "\"")
wash claims sign ./target/wasm32-unknown-unknown/release/table_tennis.wasm --name "Table Tennis" --http_server --subject $TABLE_TENNIS_SUBJECT
wash claims inspect ./target/wasm32-unknown-unknown/release/table_tennis_s.wasm
wash keys list
make release
cp ./target/wasm32-unknown-unknown/release/table_tennis_s.wasm ../

## Building and signing httpserver provider
cd ../provider-http-server
cargo build --release
wash par create --help
# Be sure to change this arch and binary flag to your architecture and shared object
wash par create --arch x86_64-macos --binary ./target/release/libwasmcloud_httpserver.dylib --capid wasmcloud:httpserver --name "HTTP Server" --vendor "wash101 developer" --compress
wash keys list
wash par inspect ./libwasmcloud_httpserver.par.gz
cp ./libwasmcloud_httpserver.par.gz ../

## Pushing resources to local registry
cd ..
docker-compose up -d 
wash reg push --help
wash reg push localhost:5000/tabletennis:0.1.0 ./table_tennis_s.wasm --insecure 
wash reg push localhost:5000/httpserver:0.1.0 ./libwasmcloud_httpserver.par --insecure
wash reg pull localhost:5000/httpserver:0.1.0 --insecure
diff httpserver.par.gz ./libwasmcloud_httpserver.par.gz

## REPL (Actor and provider ids will differ due to autogenerated keys)
wash up
ctl get hosts
ctl get inventory <HOST_ID>
ctl start actor localhost:5000/tabletennis:0.1.0
ctl call <ACTOR_ID> HandleRequest {"method": "POST", "path": "/", "body": "ping", "queryString":"", "header":{}}
ctl call <ACTOR_ID> HandleRequest {"method": "POST", "path": "/", "body": "bling", "queryString":"", "header":{}}
ctl start provider localhost:5000/httpserver:0.1.0
ctl get inventory <HOST_ID>
ctl link <ACTOR_ID> <PROVIDER_ID> wasmcloud:httpserver PORT=8080

## Separate terminal
# Bad Request
curl localhost:8080 -v
# pong
curl localhost:8080 -d "ping"

## Cleanup
rm *.par.gz *.wasm  
docker-compose down
```

