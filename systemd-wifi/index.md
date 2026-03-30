NetworkManager는 편리하지만 무겁습니다. systemd를 기반으로 한
Arch Linux 환경에서는 `systemd-networkd`와 `wpa_supplicant`만으로도
충분히 안정적인 Wi-Fi 연결을 구성할 수 있습니다.

> **참고:** 이 글은 **Arch Linux**와 **systemd 255+**를 기준으로 작성되었습니다.
> 다른 배포판에서는 경로나 명칭이 다를 수 있습니다.

## 사전 준비

필요한 패키지가 설치되어 있는지 확인합니다.

```bash
# 설치 여부 확인
pacman -Qs wpa_supplicant

# 미설치 시 설치
pacman -S wpa_supplicant
```

인터페이스 이름은 `ip link`로 확인합니다. 일반적으로 `wlan0` 또는 `wlp3s0` 형태입니다.

## wpa_supplicant 설정

`/etc/wpa_supplicant/wpa_supplicant-wlan0.conf` 파일을 작성합니다.

```ini
ctrl_interface=/run/wpa_supplicant
ctrl_interface_group=wheel
update_config=1

network={
    ssid="MyNetwork"
    psk="password123"
    key_mgmt=WPA-PSK
}
```

> **보안 주의:** `psk`를 평문으로 저장하는 대신
> `wpa_passphrase MyNetwork password`로 해시를 생성해 사용하는 것을 권장합니다.

## systemd-networkd 설정

`/etc/systemd/network/25-wireless.network` 파일을 작성합니다.

```ini
[Match]
Name=wlan0

[Network]
DHCP=yes
IgnoreCarrierLoss=3s
```

`IgnoreCarrierLoss=3s`는 연결이 잠시 끊겨도 바로 재연결 시도 없이 3초를 기다립니다.
로밍 환경에서 유용합니다.

## 서비스 활성화

세 서비스를 활성화합니다.

```bash
systemctl enable --now wpa_supplicant@wlan0
systemctl enable --now systemd-networkd
systemctl enable --now systemd-resolved
```

`systemd-resolved`는 DNS 해석을 담당합니다.
`/etc/resolv.conf`가 `/run/systemd/resolve/stub-resolv.conf`를 가리키도록 심볼릭 링크를 확인하세요.

```bash
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

## 연결 확인

```bash
networkctl status wlan0
resolvectl status
```

`routable` 상태가 나오면 연결 완료입니다.
문제가 발생하면 실시간 로그로 원인을 파악합니다.

```bash
journalctl -u wpa_supplicant@wlan0 -f
```

## 주요 옵션 비교

| 항목 | NetworkManager | systemd-networkd |
|------|---------------|-----------------|
| 데스크톱 통합 | 우수 | 제한적 |
| 리소스 사용 | 높음 | 낮음 |
| 서버/최소 환경 | 과함 | 적합 |
| 설정 방식 | GUI / nmcli | 설정 파일 |
| WPA3 지원 | 완전 | wpa_supplicant 위임 |

## 부록: 신호 감쇠 공식

자유 공간 경로 손실(Free-Space Path Loss)은 거리와 주파수에 따라 달라집니다.

**FSPL(dB) = 20 log₁₀(d) + 20 log₁₀(f) + 20 log₁₀(4π/c)**

여기서 `d`는 거리(m), `f`는 주파수(Hz), `c`는 빛의 속도(3×10⁸ m/s)입니다.
2.4 GHz Wi-Fi는 5 GHz 대역보다 같은 거리에서 경로 손실이 낮습니다.
