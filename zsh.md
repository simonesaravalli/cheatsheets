## Install and configure ZSH and oh-my-zsh on Linux

1. Install zsh

On Ubuntu:

```
apt-get install zsh
```

On Centos:

```
yum install zsh
```

2. Install oh-my-zsh via curl

https://ohmyz.sh/#install

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

3. Install ZSH Autosuggestions plugin

https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

Add it to plugins list in .zshrc

plugins=( 
    # other plugins...
    zsh-autosuggestions
)

4. Install ZSH Syntax Highlightning plugin

https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
source ./zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

5. To view changes immediately, source .zshrc

```
source ~/.zshrc
```