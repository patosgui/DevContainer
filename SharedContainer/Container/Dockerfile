FROM ubuntu:22.04

# Base packages
RUN apt-get update -o Acquire::Check-Valid-Until=false
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y \
    ccache wget unzip \
    gnupg2 lld zip \
    g++ \ 
    graphviz git cmake ninja-build \
    python3 python3-venv pip tree \
    xz-utils zip locales gpg \
    lsb-release software-properties-common gnupg \
    openssh-server ssh sudo curl htop gzip cmake-curses-gui

# Start ssh server
RUN mkdir -p /run/sshd

# Install lazygit
RUN LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*') && curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"
RUN tar xf lazygit.tar.gz lazygit
RUN install lazygit /usr/local/bin
 
# Install Clang/LLVM
RUN wget https://apt.llvm.org/llvm.sh
RUN chmod +x llvm.sh
RUN ./llvm.sh 17

ENV CC clang
ENV CXX clang++
   
# Developer tools
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y vim zsh ripgrep bat fd-find fzf zoxide duf direnv

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# Install git delta in /bin
RUN mkdir -p ~/delta
RUN wget -qO- https://github.com/dandavison/delta/releases/download/0.16.5/delta-0.16.5-x86_64-unknown-linux-gnu.tar.gz | tar xvz -C ~/delta --strip-components=1
RUN cp ~/delta/delta /bin

# Install broot in /bin
RUN wget https://dystroy.org/broot/download/x86_64-linux/broot
RUN chmod o+x ./broot
RUN cp ./broot /bin

ARG USER
ARG GIT_NAME
ARG GIT_EMAIL

# Create new user
RUN useradd -ms /bin/bash ${USER}

# Install oh my zsh
USER ${USER}
RUN sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# Do not bind ~/ between host and container. Some tools only work in specific
# OSes
COPY --chown=${USER} .gitconfig /home/${USER}
COPY --chown=${USER} .zshrc /home/${USER}

# This needs to be here because broot --install modifies ~/.zshrc
RUN broot --install
# Set "br" as installed
# https://dystroy.org/broot/install-br/#:~:text=broot%20%2D%2Dset%2Dinstall%2Dstate,values%20are%20undefined%20and%20refused%20).
RUN broot --set-install-state installed

# Configure git globally according to credentials
RUN git config --global user.email "${GIT_EMAIL}"
RUN git config --global user.name "${GIT_NAME}"

