## 首先安装LazyVim
Install the [LazyVim Starter](https://github.com/LazyVim/starter)

- Make a backup of your current Neovim files:
    
    ```shell
    mv ~/.config/nvim ~/.config/nvim.bak
    mv ~/.local/share/nvim ~/.local/share/nvim.bak
    ```
    
- Clone the starter
    
    ```shell
    git clone https://github.com/LazyVim/starter ~/.config/nvim
    ```
    
- Remove the `.git` folder, so you can add it to your own repo later
    
    ```shell
    rm -rf ~/.config/nvim/.git
    ```
    
- Start Neovim!
    
    ```shell
    nvim
    ```

