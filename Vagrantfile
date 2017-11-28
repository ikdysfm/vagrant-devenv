# -*- mode: ruby -*-
# vi: set ft=ruby :

# 必須プラグインの自動インストール
# ~/.vagrant.d/Vagrantfileに分離して共通化しても良いかも
required_plugins = %w(sahara vagrant-vbguest vagrant-cachier)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # ネットワーク設定は大きく3種類。それぞれ長所短所があり、併用も可能
  #
  # Forwarded Ports
  # ホストの8080番ポートへのアクセスをゲストの80番ポートにフォワードする例
  # host_ip, guest_ipはipを限定するものでデフォルトは'empty'らしい。詳細な挙動についてはちゃんと調べる
  # 自動衝突検知からのポート番号変更(範囲指定可)も可能で、UDPも指定できる
  # 通すべきポートが多いと明示的に指定するのが面倒という短所がある
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  #
  # Private Networks
  # ホストと(複数の)ゲスト間でのみ通信可能なネットワークを構築する
  # 隔離されているのが長所であり短所
  # webサーバ、dbサーバなど複数マシンを立ててテストするならこれが向いている
  # config.vm.network "private_network", ip: "192.168.33.10"
  #
  # Public Networks
  # 独立した物理マシンのように振る舞うが、(今のところ)静的なIPを指定できない
  # config.vm.network "public_network"

  # 共有フォルダ設定。デフォルトだと ".", "/vagrant"
  # config.vm.synced_folder "../host_dir", "/guest_dir", option

  # ホストの秘密鍵を利用する
  # ホスト側でssh-agentを起動しておく必要がある
  # とりあえず都度ゲストで鍵を作ることにする
  # config.ssh.forward_agent = true

  # キャッシュ設定
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box # box単位かmachine単位か
    # config.cache.synced_folder_opts = { 
      # type: :nfs, # ホストがwindowsだと対応してない
      # # The nolock option can be useful for an NFSv3 client that wants to avoid the
      # # NLM sideband protocol. Without this option, apt-get might hang if it tries
      # # to lock files needed for /var/cache/* operations. All of this can be avoided
      # # by using NFSv4 everywhere. Please note that the tcp option is not the default.
      # mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    # }
  end
  
  # プロバイダ固有の設定。詳しくはドキュメントを見る
  config.vm.provider "virtualbox" do |vb|
    config.vbguest.auto_update = true # vbguestプラグインが必要
    config.vbguest.no_remote = false # リモートからダウンロードする
    vb.name = "xubuntu"
    vb.gui = true
    vb.cpus = 2
    vb.memory = "4096"
    vb.customize [
      # https://www.virtualbox.org/manual/ch08.html
      "modifyvm", :id,
      "--clipboard", "bidirectional", # クリップボードの共有
      "--draganddrop", "bidirectional", # ドラッグ&ドロップ
      "--chipset", "ich9", # チップセット
      "--ioapic", "on", # I/O APICを有効化
      "--rtcuseutc", "on", # ハードウェアクロックをUTCにする
      "--cpuexecutioncap", "100", # 使用率制限
      "--pae", "on", # PAE/NXを有効化
      "--paravirtprovider", "hyperv", # 準仮想化インタフェース
      "--hwvirtex", "on", # VT-x/AMD-Vを有効化
      "--nestedpaging", "on", # ネステッドページングを有効化
      "--largepages", "on", # ハイパーバイザがlargepageを使用する
      "--vram", "256", # ビデオメモリー
      "--accelerate3d", "on", # 3Dアクセラレーションを有効化
    ]
  end

  # v1.3より、初回up時以外は明示的に指定しないとプロビジョニングは行われない
  # config.vm.provision "shell", inline: <<-SHELL
    # privilegedオプション未指定だとrootで実行される
    # if [ -f "/var/vagrant_provision" ]; then
      # exit 0 # 一回限りの実行とする
    # fi
    # aptの利用リポジトリを日本サーバーに変更する
    # sed -i.bak -e "s%http://[^ ]\+%http://ftp.jaist.ac.jp/pub/Linux/ubuntu/%g" /etc/apt/sources.list
    # touch /var/vagrant_provision
  # SHELL

  config.vm.provision "ansible_local" do |ansible|
    # https://www.vagrantup.com/docs/provisioning/ansible_common.html
    ansible.compatibility_mode = "2.0"
    ansible.provisioning_path = "/vagrant/provisioning" # cfgファイル置き場であり、inventory_pathとplaybookはここからの相対パスとなる
    ansible.inventory_path = "hosts"
    ansible.playbook = "playbook.yml"
    ansible.limit = "localhost" # ansibleのhostsと一致させる必要がある。省略すると仮想マシン名を指定したことになる
    # ansible.verbose = "v"
  end

  # vagrant-rebootを入れようかと思ったが保留。初回のGUI環境インストール後は手動で再起動することにする
end
