pkgname='d-utils'
pkgver="0.0.0"
pkgdesc='utils to run docker images without docker'
arch=('all')
license='MIT'
depends=('curl' 'tar' 'jq' 'bubblewrap' 'python3')

package() {
	install -m 755 -D d-pull "$pkgdir/usr/bin/d-pull"
	install -m 755 -D d-run "$pkgdir/usr/bin/d-run"
}
