1. Необходимо удалить сетевой драйвер с упоминанием слова "TUN" через "Диспетчер устройств"
2. Выполнить в консоли команды:
```shell
netsh winsock reset
netsh int ip reset
netsh winhttp reset proxy
ipconfig /flushdns
ipconfig /release
ipconfig /renew
```
3. Перезагрузить компьютер