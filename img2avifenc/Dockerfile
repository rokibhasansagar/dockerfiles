# syntax=docker/dockerfile:1-labs
FROM fr3akyphantom/archy-base:latest

# USER app

# WORKDIR /tmp

SHELL ["/bin/bash", "-c"]

RUN --mount=type=secret,id=RCLONE_CONFIG_HASH,uid=1000 <<-'EOL'
	# Update System Immediately
	( sudo pacman -Syu --noconfirm 2>/dev/null ) || ( sudo pacman -Syu --noconfirm 2>/dev/null || true )
	( sudo rm -rvf /var/cache/pacman/pkg/*.pkg.tar.zst* 2>/dev/null || true )
	set -ex
	export PATH="/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/bin"
	# Setup rclone and unzip
	sudo pacman -S --noconfirm --needed rclone unzip
	# Setup personal rclone config
	export RCLONE_CONFIG_HASH=$(< /run/secrets/RCLONE_CONFIG_HASH) || true
	mkdir -p ~/.config/rclone
	curl -sL --retry 5 --retry-connrefused "https://gist.github.com/rokibhasansagar/${RCLONE_CONFIG_HASH}" >./rcl.log
	curl -sL --retry 5 --retry-connrefused "https://gist.github.com/rokibhasansagar/${RCLONE_CONFIG_HASH}/$(grep -m1 -o "archive/.*.zip" ./rcl.log)" -O
	unzip -j *.zip && cat *.rclone.config >~/.config/rclone/rclone.conf
	rm *.zip ./rcl.log *.rclone.config
	# Install aom (psy101-git)
	rclone copy "ms365:Public/ArchVapourBuilds/native/Step_Zero2/" --include="aom-psy101-git-*.pkg.tar.zst" . --fast-list -v 2>/dev/null
	sudo pacman -U --noconfirm aom-psy101-git-*.pkg.tar.zst
	sudo sed -i '/^#IgnorePkg/c\IgnorePkg = aom' /etc/pacman.conf 2>/dev/null
	# Install imagemagick
	sudo pacman -S --noconfirm --needed imagemagick libavif
	# Install others
	sudo pacman -S --noconfirm --needed mediainfo vmaf
	if command -v avifenc 2>/dev/null; then avifenc --version; fi
	# Cleanup
	sudo rm -rvf ~/.config/rclone/rclone.conf
	( sudo rm -rvf /tmp/* /var/lib/pacman/sync/* /var/cache/pacman/pkg/* /home/app/.cache/yay/* /home/app/.cache/paru/* 2>/dev/null || true )
EOL

VOLUME ["/data"]
WORKDIR /data

ENTRYPOINT [ "/usr/bin/bash" ]
