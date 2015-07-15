# Checkout and build ALE server

cd ..
git clone https://github.com/mgbellemare/Arcade-Learning-Environment.git (fetch)
cd Arcade-Learning-Environment
sudo apt-get install libsdl1.2-dev
cmake -DUSE_SDL=ON -DUSE_RLGLUE=OFF -DBUILD_EXAMPLES=ON .
make -j 4

# Run server in FIFO mode
./install_dependencies.sh

cd torch

patch -p1 < ../torch.patch

cd -


mkfifo ale_fifo

nc -l -p 1567 < ale_fifo | ./ale -game_controller fifo -display_scree true -run_length_encoding false ../DeepMind-Atari-Deep-Q-Learner/roms/breakout.bin > ale_fifo

# To play on remote server - set ALE_HOST enviroment variable
env ALE_HOST=localhost ./run_cpu breakout
env ALE_HOST=localhost ./run_gpu breakout

# To play localy - ommit it
./run_cpu breakout
./run_gpu breakout

#or even
env -u ALE_HOST ./run_cpu breakout 
