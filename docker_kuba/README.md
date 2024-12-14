
docker build -t oiio_ci .

$ipV4 = Test-Connection -ComputerName (hostname) -Count 1  | Select -ExpandProperty IPV4Address
$ipV4 = $ipV4.IPAddressToString
docker run --rm -it -e "DISPLAY=${ipV4}:0" --entrypoint /bin/bash oiio_ci

## Run tests:
PYTHONPATH=$(pwd)/lib/python/site-packages:$PYTHONPATH ctest -E broken --force-new-ctest-process --output-on-failure


docker image prune -a
docker volume prune -a

docker rmi -f $(docker images -a --filter=dangling=true -q)
docker rm $(docker ps --filter=status=exited --filter=status=created -q)

ctest -C Release -E broken --force-new-ctest-process --output-on-failure


187/187 Test #187: unit_compute ............................   Passed    0.69 sec

99% tests passed, 2 tests failed out of 187

Total Test time (real) = 396.87 sec

The following tests FAILED:
          1 - cmake-consumer (Failed)
          3 - docs-examples-cpp (Failed)