# Playing on remote ALE server

Checkout and build ALE server
```
cd ..
git clone https://github.com/mgbellemare/Arcade-Learning-Environment.git (fetch)
cd Arcade-Learning-Environment
sudo apt-get install libsdl1.2-dev
cmake -DUSE_SDL=ON -DUSE_RLGLUE=OFF -DBUILD_EXAMPLES=ON .
make -j 4
```

Install dependencies and apply the patch from this repo
```
./install_dependencies.sh

cd torch
patch -p1 < ../torch.patch
cd -
```

Run server in FIFO mode
```
mkfifo ale_fifo

nc -l -p 1567 < ale_fifo | ./ale -game_controller fifo -display_screen true -run_length_encoding false ../DeepMind-Atari-Deep-Q-Learner/roms/breakout.bin > ale_fifo
```

# To play on remote server - set ALE_HOST enviroment variable
NB! Plaese read aout setup ALE server setup: https://github.com/gerrich/ale_team_runner
```
# train at ale_server on local machine
env ALE_LOGIN=test ALE_PASS=test12 ALE_HOST=localhost ALE_PORT=1567 ./run_cpu breakout
env ALE_LOGIN=test ALE_PASS=test12 ALE_HOST=localhost ALE_PORT=1567 ./run_gpu breakout

# train at ale_server on tesing server (passwords are mailed to teams)
env ALE_LOGIN=team_15 ALE_PASS=XXXX ALE_HOST=93.175.18.243 ALE_PORT=17015 ./run_cpu breakout
```

To play localy - ommit it
```
./run_cpu breakout
./run_gpu breakout
```
or even

```env -u ALE_HOST ./run_cpu breakout ```

## Консольный клиент можно запустить так:
```
mkfifo client_fifo
nc localhost 1567 < client_fifo | ./simple > client_fifo
```

если хочется сохранить входноый/выходной потоки, можно использовать tee:
```nc localhost 1567 < client_fifo |tee simple_in | ./simple | tee simple_out > client_fifo```

аналогично можно сделать на сервере:
```
nc -l 1567 < ale_fifo | tee ale_in | ./ale -game_controller fifo -display_screen true -run_length_encoding false ../DeepMind-Atari-Deep-Q-Learner/roms/breakout.bin | tee ale_out > ale_fifo
```

чтобы все работало, клиент должен отдавать stdout без буферизации:
на с++ можно явно делать fflush(stdout);
```
$ cat simple.cpp
#include <cstdio>

int main()
{
  const int sz = 128*1024;
  char s[sz];

  fgets(s, sz, stdin);
  fprintf(stdout, "0,0,0,1\n");
  fflush(stdout);

  for(int i = 0; i < 1000*1000; ++i) {
    fgets(s, sz, stdin);
    fprintf(stdout, "12,18\n");
    fflush(stdout);
  }
  return 0;
}
```
