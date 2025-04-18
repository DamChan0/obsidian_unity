```bash 
sudo apt update && sudo apt upgrade -y

sudo apt install -y curl git libwebkit2gtk-4.0-dev build-essential

# node js 
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# check nodejs install
node -v
npm -v

# install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# check rust 
rustc --version
cargo --version

# isntall Tauri 

cargo install tauri-cli

```