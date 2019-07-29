# coding: utf-8

from fabric.api import run, env, task, parallel

env.user = 'root'
env.password = 'your_password'
env.hosts = [
'hostname_or_ip_address'
]

@task
@parallel
# ファイル数の確認（ファイル数は152を表示する想定）
def count_files():
    run('ls /var/www/movie_dir/movie | wc -l')

@task
@parallel
# 作業第1弾　/var/www/movie_dir/movie配下にある既存動画（*.mp4）を
# 新ディレクトリ/mnt/new_disk/movie_dir配下へコピー
def file_copy():
	run('mkdir /mnt/new_disk/movie && cp /var/www/movie_dir/movie/*.mp4 /mnt/new_disk/movie/')

@task
@parallel
# 作業第1段コピー状態の確認
def check_result():
    run('ls /mnt/new_disk/movie/ | wc -l')

@task
@parallel
# 作業第1段コピーでコピーしたaaaで始まるファイルのコピー結果を確認
def count_BUK():
    run('ls /mnt/new_disk/movie/aaa*.mp4 | wc -l')

@task
@parallel
# 作業第1段コピーでコピーしたbbbで始まるファイルのコピー結果を確認
def count_ART():
    run('ls /mnt/new_disk/movie/bbb*.mp4 | wc -l')

@task
@parallel
# 作業第2弾　AWS_S3のmovie配下に保存されれている追加動画をS3から新ディレクトリ/mnt/new_disk/movie/配下へコピー
#
def s3_cp_from_list1():
	for line in open('command_list', 'r'):
		run(line, warn_only=True)

# 追加動画用の新設定ファイルを転送
@task
@parallel
def copy_newpd():
	run('aws s3 cp s3://movie_bukete/new_movies /mnt/new_disk/movie/ --recursive')

# PDの新設定ファイルを転送
@task
@parallel
def check_result_newpd():
	run('ls -l /mnt/new_disk | grep 'movie)

@task
@parallel
def switch_newpd():
	run('mv /mnt/new_disk/movie /mnt/new_disk/movie.old && mv /mnt/new_disk/movie/ /var/www/movie')
