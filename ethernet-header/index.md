# Ethernet Header 분석

'Ethernet'은 뜻이 하나로 정해진 용어 같지만, 실제로는 유선 LAN 통신에 관여하는 HW/SW 규격 전체를 넓게 의미하는 말이다. IEEE 802.3, a.k.a 'IEEE Standard for Ethernet'에 표준규격이 문서화되어 있다[^1].

## Ethernet Frame

Ethernet 프레임과 Ethernet 헤더는 구별해야 한다. 프레임은 MAC 계층이 주고받는 단위이고, 헤더는 그 프레임의 맨 앞 14바이트이다. 즉, 프레임이 큰 틀이고 헤더는 그 안의 일부이다. 'Ethernet 헤더'와 'Ethernet 프레임 헤더'는 같은 것을 가리키며, 리눅스 커널의 `struct ethhdr`([`include/uapi/linux/if_ether.h:164`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/uapi/linux/if_ether.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n164)) 주석도 이 헤더를 'Ethernet frame header'라고 부른다[^2]. 프레임의 전체 구조는 다이어그램 1과 같다.

```canvas
{"label": "An Ethernet frame on the wire: a 7-octet preamble and a 1-octet SFD come before the MAC frame of 64 to 1518 octets, whose first 14 octets (destination, source and Length/Type) are the Ethernet header, followed by 46 to 1500 octets of data and a 4-octet FCS",
 "items": [
  {"row": "wire", "x": 0, "y": 60, "h": 56, "cells": [
   {"w": 104, "label": ["Preamble", "7"], "fill": "gap"},
   {"w": 64, "label": ["SFD", "1"], "fill": "gap"},
   {"w": 124, "label": ["Destination", "6"], "fill": "purple"},
   {"w": 96, "label": ["Source", "6"], "fill": "purple"},
   {"w": 124, "label": ["Length/Type", "2"], "fill": "purple"},
   {"w": 124, "label": ["Data", "46-1500"]},
   {"w": 64, "label": ["FCS", "4"]}
  ]},
  {"line": ["wire|2.top", [168, 16]]},
  {"line": ["wire|5.top", [512, 16]]},
  {"dimension": [[168, 24], [512, 24]], "label": "Ethernet header (14 bytes)"},
  {"line": ["wire|0.bottom", [0, 204]]},
  {"line": ["wire|2.bottom", [168, 160]]},
  {"line": ["wire|7.bottom", [700, 204]]},
  {"dimension": [[168, 152], [700, 152]], "label": "MAC frame (64-1518 bytes)", "side": "below"},
  {"dimension": [[0, 196], [700, 196]], "label": "on the wire (72-1526 bytes)", "side": "below"}
 ]}
```

프레임 앞에 7바이트 preamble과 1바이트 SFD(Start Frame Delimiter)가 붙는다[^3][^4]. preamble은 받는 쪽 네트워크 장비의 물리 계층 회로가 장비로 들어오는 프레임의 타이밍에 동기를 맞추도록 하는 필드이다[^3]. preamble의 비트열은 `10101010`이 7번 연속으로 이어진 것이고, SFD는 `10101011`이다[^4]. 두 비트열은 LSB부터, 즉 MAC이 링크로 내보내는 순서대로 적은 것이다([RFC 9542 1.1절](https://www.rfc-editor.org/rfc/rfc9542#section-1.1))[^4][^5]. 바이트 값으로 읽으면 preamble의 각 바이트는 `0x55`, SFD는 `0xD5`이며, 비트열은 다이어그램 2와 같다.

```canvas
{"label": "The preamble and SFD in MAC transmit order: seven octets of 10101010 (0x55), octets 3 to 6 elided, then the SFD 10101011 (0xD5), each written from its LSB, the first bit sent",
 "items": [
  {"row": "pre", "x": 0, "y": 60, "h": 56, "cells": [
   {"w": 120, "label": ["10101010", "0x55"]},
   {"w": 120, "label": ["10101010", "0x55"]},
   {"w": 104, "label": "...", "line": "dashed"},
   {"w": 120, "label": ["10101010", "0x55"]},
   {"w": 144, "label": ["10101011", "0xD5"], "fill": "purple"}
  ]},
  {"line": ["pre|0.top", [0, 16]]},
  {"line": ["pre|4.top", [464, 16]]},
  {"line": ["pre|5.top", [608, 16]]},
  {"dimension": [[0, 24], [464, 24]], "label": "Preamble (7 bytes)"},
  {"dimension": [[464, 24], [608, 24]], "label": "SFD (1 byte)"},
  {"arrow": [[0, 152], [608, 152]], "label": {"text": "transmit order, LSB first", "side": "below"}}
 ]}
```

다이어그램 2의 비트열이 케이블 위 신호를 그대로 나타내지는 않는다. 비트열을 전기 신호로 바꾸는 일은 물리 계층(PHY)이 맡으며, 바꾸는 방식은 Ethernet 규격마다 다르다. 예를 들어 10 Mb/s의 10BASE-T는 비트를 Manchester 부호로 보내며, 비트마다 전압이 높은 쪽에서 낮은 쪽으로 바뀌면 0, 낮은 쪽에서 높은 쪽으로 바뀌면 1이다[^6]. preamble의 마지막 바이트와 SFD를 이 부호로 보내면 다이어그램 3과 같다.

```canvas
{"label": "The last preamble byte 10101010 and the SFD 10101011 in transmit order, as bits, as a data level and as a 10BASE-T Manchester signal, where a 0 falls and a 1 rises in the middle of its bit, so the alternating preamble changes only mid-bit and the final 11 of the SFD adds a change at a bit boundary",
 "items": [
  {"line": [[0, 16], [0, 40]]},
  {"line": [[288, 16], [288, 40]]},
  {"line": [[576, 16], [576, 40]]},
  {"dimension": [[0, 24], [288, 24]], "label": "Preamble 0x55"},
  {"dimension": [[288, 24], [576, 24]], "label": "SFD 0xD5"},
  {"line": [[0, 70], [0, 186]], "dash": true, "weight": "marker"},
  {"line": [[36, 70], [36, 186]], "dash": true, "weight": "marker"},
  {"line": [[72, 70], [72, 186]], "dash": true, "weight": "marker"},
  {"line": [[108, 70], [108, 186]], "dash": true, "weight": "marker"},
  {"line": [[144, 70], [144, 186]], "dash": true, "weight": "marker"},
  {"line": [[180, 70], [180, 186]], "dash": true, "weight": "marker"},
  {"line": [[216, 70], [216, 186]], "dash": true, "weight": "marker"},
  {"line": [[252, 70], [252, 186]], "dash": true, "weight": "marker"},
  {"line": [[288, 70], [288, 186]], "dash": true, "weight": "marker"},
  {"line": [[324, 70], [324, 186]], "dash": true, "weight": "marker"},
  {"line": [[360, 70], [360, 186]], "dash": true, "weight": "marker"},
  {"line": [[396, 70], [396, 186]], "dash": true, "weight": "marker"},
  {"line": [[432, 70], [432, 186]], "dash": true, "weight": "marker"},
  {"line": [[468, 70], [468, 186]], "dash": true, "weight": "marker"},
  {"line": [[504, 70], [504, 186]], "dash": true, "weight": "marker"},
  {"line": [[540, 70], [540, 186]], "dash": true, "weight": "marker"},
  {"line": [[576, 70], [576, 186]], "dash": true, "weight": "marker"},
  {"text": "1", "x": 18, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 54, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 90, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 126, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 162, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 198, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 234, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 270, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 306, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 342, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 378, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 414, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 450, "y": 56, "anchor": "middle"},
  {"text": "0", "x": 486, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 522, "y": 56, "anchor": "middle"},
  {"text": "1", "x": 558, "y": 56, "anchor": "middle"},
  {"line": [[0, 84], [36, 84], [36, 112], [72, 112], [72, 84], [108, 84], [108, 112], [144, 112], [144, 84], [180, 84], [180, 112], [216, 112], [216, 84], [252, 84], [252, 112], [288, 112], [288, 84], [324, 84], [324, 112], [360, 112], [360, 84], [396, 84], [396, 112], [432, 112], [432, 84], [468, 84], [468, 112], [504, 112], [504, 84], [576, 84]], "weight": "marker"},
  {"line": [[0, 174], [18, 174], [18, 146], [54, 146], [54, 174], [90, 174], [90, 146], [126, 146], [126, 174], [162, 174], [162, 146], [198, 146], [198, 174], [234, 174], [234, 146], [270, 146], [270, 174], [306, 174], [306, 146], [342, 146], [342, 174], [378, 174], [378, 146], [414, 146], [414, 174], [450, 174], [450, 146], [486, 146], [486, 174], [522, 174], [522, 146], [540, 146], [540, 174], [558, 174], [558, 146], [576, 146]], "weight": "marker", "tone": "purple"},
  {"text": "bits", "x": -14, "y": 56, "anchor": "end", "role": "note"},
  {"text": "data", "x": -14, "y": 98, "anchor": "end", "role": "note"},
  {"text": "Manchester", "x": -14, "y": 160, "anchor": "end", "role": "note"}
 ]}
```

반면 노트북이나 데스크톱이 주로 쓰는 1000BASE-T는 Manchester 부호를 쓰지 않는다. 1000BASE-T는 네 쌍의 선을 함께 쓰고 전압을 여러 단계로 나눠 한 번에 한 바이트씩 보낸다([802.3ab 튜토리얼 26쪽](https://grouper.ieee.org/groups/802/3/ab/pub/tutorial.pdf#page=26), [38쪽](https://grouper.ieee.org/groups/802/3/ab/pub/tutorial.pdf#page=38))[^7]. 프레임이 없을 때도 idle 신호를 계속 보낸다([48쪽](https://grouper.ieee.org/groups/802/3/ab/pub/tutorial.pdf#page=48))[^7]. 10 Mb/s보다 빠른 PHY에서는 preamble이 받는 쪽의 동기에 필요하지 않으며, 1000BASE-T PHY는 preamble의 처음 두 바이트를 SSD(Start-of-Stream Delimiter)라는 시작 기호로 바꿔 보낸다([UNH MAC 테스트 스위트 38쪽](https://www.iol.unh.edu/sites/default/files/testsuites/ethernet/MAC/MAC_Test_Suite_v5.0.pdf#page=38))[^8]. 받는 쪽 PHY는 이 자리를 다시 preamble로 바꿔 MAC에 넘긴다([30쪽](https://www.iol.unh.edu/sites/default/files/testsuites/ethernet/MAC/MAC_Test_Suite_v5.0.pdf#page=30))[^8]. 같은 구간을 1000BASE-T로 보내면 다이어그램 4와 같다.

```canvas
{"label": "The same preamble and SFD on a 1000BASE-T link: the four pairs carry idle until the frame starts, then the SSD1 and SSD2 start delimiter in place of the first two preamble bytes, and one data symbol for each remaining byte",
 "items": [
  {"row": "mac", "x": 0, "y": 60, "h": 48, "cells": [
   {"w": 72, "label": "-", "fill": "gap"},
   {"w": 72, "label": "-", "fill": "gap"},
   {"w": 72, "label": "0x55"},
   {"w": 72, "label": "0x55"},
   {"w": 72, "label": "0x55"},
   {"w": 72, "label": "...", "line": "dashed"},
   {"w": 72, "label": "0x55"},
   {"w": 72, "label": "0xD5"}
  ]},
  {"row": "phy", "x": 0, "y": 148, "h": 48, "cells": [
   {"w": 72, "label": "IDLE", "fill": "gap"},
   {"w": 72, "label": "IDLE", "fill": "gap"},
   {"w": 72, "label": "SSD1", "fill": "purple"},
   {"w": 72, "label": "SSD2", "fill": "purple"},
   {"w": 72, "label": "DATA"},
   {"w": 72, "label": "...", "line": "dashed"},
   {"w": 72, "label": "DATA"},
   {"w": 72, "label": "DATA"}
  ]},
  {"line": ["mac|2.top", [144, 16]]},
  {"line": ["mac|7.top", [504, 16]]},
  {"line": ["mac|8.top", [576, 16]]},
  {"dimension": [[144, 24], [504, 24]], "label": "Preamble (7 bytes)"},
  {"dimension": [[504, 24], [576, 24]], "label": "SFD"},
  {"text": "MAC", "x": -14, "y": 84, "anchor": "end", "role": "note"},
  {"text": "4 pairs", "x": -14, "y": 172, "anchor": "end", "role": "note"}
 ]}
```

이 값은 특수한 예가 아니다. IEEE 802.3의 MAC 프레임 구조는 10 Mb/s부터 1000 Mb/s까지 같은 preamble과 SFD 필드를 두므로[^3], 노트북이나 데스크톱의 1000BASE-T 같은 유선 Ethernet에서 MAC은 이 값을 그대로 만든다. 반면 휴대폰이 주로 쓰는 Wi-Fi는 꼬임쌍선, 광섬유 같은 유선 매체를 다루는 802.3의 물리 계층이 아니므로 이 값과 관계가 없다[^1]. 다만 모든 Ethernet 규격과 장비에서 preamble과 SFD가 항상 이 값으로만 구성되는 것은 아니다([`Documentation/networking/phy.rst:311`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Documentation/networking/phy.rst?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n311), `PHY_INTERFACE_MODE_QUSGMII`)[^2][^4].

이 8바이트는 MAC 프레임에 들어가지 않는다. 다이어그램 1에서 선 위 비트열의 0~7번째 바이트가 preamble과 SFD이고, MAC 프레임은 실제 장비로 들어온 바이트열의 8번째 바이트부터 시작한다. IEEE 802.3의 MAC 의사코드에서는 MAC 계층이 보낼 때 프레임 앞에 이 8바이트를 먼저 내보내고, 받을 때는 이 8바이트를 떼어 낸 뒤 프레임을 읽는다[^4]. 따라서 커널을 포함한 소프트웨어는 preamble과 SFD를 보지 못하며, 커널의 프레임 길이 상수도 preamble을 빼고 센다(인용 4).

MAC 프레임은 Destination MAC address에서 시작해 FCS(Frame Check Sequence)로 끝난다[^4]. 태그가 없는 프레임의 길이는 최소 64바이트, 최대 1518바이트이며[^9][^10], glibc는 이 두 값을 `ETHER_MIN_LEN`과 `ETHER_MAX_LEN`으로 정의한다[^11]. Length/Type 다음의 데이터 필드는 최소 46바이트이고, 데이터가 이보다 짧으면 패딩을 덧붙여 최소 길이를 맞춘다[^12][^4]. 데이터 필드의 최대 길이는 1500바이트이다(인용 4의 `ETH_DATA_LEN`).

마지막 FCS는 Destination MAC address부터 패딩까지를 덮는 32비트 CRC 값이다[^4]. 보내는 쪽 MAC은 이 범위의 CRC를 계산해 FCS에 넣고, 받는 쪽 MAC은 받은 프레임의 CRC를 다시 계산해 FCS와 다르면 그 프레임을 오류(`frameCheckError`)로 처리한다[^4]. 즉, FCS는 받는 쪽이 프레임을 검사하는 데 쓰는 값으로 프레임 맨 끝에 붙으며, 맨 앞 14바이트인 Ethernet 헤더에는 속하지 않는다(인용 4의 `struct ethhdr`).

리눅스는 받은 프레임을 FCS를 뺀 채 올리고, `NETIF_F_RXFCS`([`include/linux/netdev_features.h:67`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/linux/netdev_features.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n67), `NETIF_F_RXFCS_BIT`)를 켠 장치만 FCS를 데이터 뒤에 붙인다[^2]. FCS가 맞지 않는 프레임은 장치가 버리며, `NETIF_F_RXALL`을 켠 장치만 이런 프레임까지 받는다([`Documentation/networking/netdev-features.rst:155`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Documentation/networking/netdev-features.rst?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n155))[^2]. 이 동작은 장치 드라이버가 구현하므로 세부 방식은 드라이버마다 다르다. Intel E800 계열 NIC의 ice 드라이버 기준으로 보면, ice는 수신 큐를 설정할 때 `ICE_RX_FLAGS_CRC_STRIP_DIS` 플래그가 없으면 큐 문맥의 `crcstrip`을 켜서 NIC가 FCS를 뗀 뒤 프레임을 호스트 메모리에 쓰게 한다([`drivers/net/ethernet/intel/ice/ice_base.c:507`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_base.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n507), `ice_setup_rx_ctx()`). `NETIF_F_RXFCS`를 켜면 `ice_set_features()`가 `ice_vsi_cfg_crc_strip()`으로 모든 수신 링에 이 플래그를 세우고 큐를 다시 시작하므로, 그 뒤로는 프레임 데이터 뒤에 FCS 4바이트가 남는다([`drivers/net/ethernet/intel/ice/ice_main.c:6541`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_main.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n6541), [`drivers/net/ethernet/intel/ice/ice_lib.c:1482`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_lib.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n1482))[^2].

FCS를 실제로 떼고 프레임을 호스트 메모리에 쓰는 일은 커널 코드가 아니라 NIC 하드웨어가 한다. 드라이버는 `ice_write_rxq_ctx()`로 수신 큐 문맥을 하드웨어 형식으로 묶어 NIC의 `QRX_CONTEXT` 레지스터에 써 넣을 뿐이며, `crcstrip`은 이 문맥의 117번째 비트이다([`drivers/net/ethernet/intel/ice/ice_base.c:569`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_base.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n569), [`drivers/net/ethernet/intel/ice/ice_common.c:1433`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_common.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n1433), [`drivers/net/ethernet/intel/ice/ice_common.c:1393`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_common.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n1393)). 받는 경로의 `ice_clean_rx_irq()`도 NIC가 수신 디스크립터에 적어 준 길이를 그대로 쓰고, FCS 4바이트를 따로 빼지 않는다([`drivers/net/ethernet/intel/ice/ice_txrx.c:960`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/net/ethernet/intel/ice/ice_txrx.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n960))[^2]. v6.18 ice 드라이버의 해당 부분은 인용 1, 인용 2, 인용 3과 같다.

<!-- source-ref id="ice_set_features" -->
```c
static int
ice_set_features(struct net_device *netdev, netdev_features_t features)
{
	netdev_features_t changed = netdev->features ^ features;

/* ... */

	/* Turn on receive of FCS aka CRC, and after setting this
	 * flag the packet data will have the 4 byte CRC appended
	 */
	if (changed & NETIF_F_RXFCS) {
		if ((features & NETIF_F_RXFCS) &&
		    (features & NETIF_VLAN_STRIPPING_FEATURES)) {
			dev_err(ice_pf_to_dev(vsi->back),
				"To disable FCS/CRC stripping, you must first disable VLAN stripping\n");
			return -EIO;
		}

		ice_vsi_cfg_crc_strip(vsi, !!(features & NETIF_F_RXFCS)); /* [!hl] */
		ret = ice_down_up(vsi);
		if (ret)
			return ret;
	}
```

<!-- source-ref id="ice_setup_rx_ctx" -->
```c
static int ice_setup_rx_ctx(struct ice_rx_ring *ring)
{

/* ... */

	/* Strip the Ethernet CRC bytes before the packet is posted to host
	 * memory.
	 */
	rlan_ctx.crcstrip = !(ring->flags & ICE_RX_FLAGS_CRC_STRIP_DIS); /* [!hl] */

/* ... */

	/* Absolute queue number out of 2K needs to be passed */
	err = ice_write_rxq_ctx(hw, &rlan_ctx, pf_q); /* [!hl] */
```

<!-- source-ref id="ice_write_rxq_ctx" -->
```c
static void ice_copy_rxq_ctx_to_hw(struct ice_hw *hw,
				   const ice_rxq_ctx_buf_t *rxq_ctx,
				   u32 rxq_index)
{
	/* Copy each dword separately to HW */
	for (int i = 0; i < ICE_RXQ_CTX_SIZE_DWORDS; i++) {
		u32 ctx = ((const u32 *)rxq_ctx)[i];

		wr32(hw, QRX_CONTEXT(i, rxq_index), ctx); /* [!hl] */

		ice_debug(hw, ICE_DBG_QCTX, "qrxdata[%d]: %08X\n", i, ctx);
	}
}

/* ... */

/* LAN Rx Queue Context */
static const struct packed_field_u8 ice_rlan_ctx_fields[] = {
				 /* Field		Width	LSB */

/* ... */

	ICE_CTX_STORE(ice_rlan_ctx, crcstrip,		1,	117), /* [!hl] */

/* ... */

int ice_write_rxq_ctx(struct ice_hw *hw, struct ice_rlan_ctx *rlan_ctx,
		      u32 rxq_index)
{
	ice_rxq_ctx_buf_t buf = {};

	if (rxq_index > QRX_CTRL_MAX_INDEX)
		return -EINVAL;

	ice_pack_rxq_ctx(rlan_ctx, &buf);
	ice_copy_rxq_ctx_to_hw(hw, &buf, rxq_index);

	return 0;
}
```

커널의 길이 상수 `ETH_ZLEN`(60)과 `ETH_FRAME_LEN`(1514)도 FCS를 뺀 값이다(인용 4). PHY 계층 및 Ethernet 표준 등에 관련된 서술이 아닌 만큼, 이 부분에 대한 더 상세한 서술은 생략한다.

## Ethernet 헤더

Ethernet 헤더는 14바이트로 이루어진 단순한 구조이다. 순서대로 Destination MAC address 6바이트, Source MAC address 6바이트, EtherType 2바이트의 세 필드로 나뉜다(인용 4). 레이아웃은 다이어그램 5와 같다.

```canvas
{"label": "struct ethhdr, the 14-byte Ethernet header: h_dest holds bits 0 to 47, h_source bits 48 to 95, and h_proto, a big-endian 16-bit EtherType, bits 96 to 111",
 "items": [
  {"row": "hdr", "x": 0, "y": 60, "h": 56, "cells": [
   {"w": 240, "label": ["h_dest", "unsigned char[6]"], "fill": "purple"},
   {"w": 240, "label": ["h_source", "unsigned char[6]"], "fill": "purple"},
   {"w": 140, "label": ["h_proto", "__be16"], "fill": "purple"}
  ]},
  {"line": ["hdr|0.top", [0, 16]]},
  {"line": ["hdr|1.top", [240, 16]]},
  {"line": ["hdr|2.top", [480, 16]]},
  {"line": ["hdr|3.top", [620, 16]]},
  {"dimension": [[0, 24], [240, 24]], "label": "bits 0-47"},
  {"dimension": [[240, 24], [480, 24]], "label": "bits 48-95"},
  {"dimension": [[480, 24], [620, 24]], "label": "bits 96-111"}
 ]}
```

리눅스는 이 헤더를 UAPI 헤더 `include/uapi/linux/if_ether.h`의 `struct ethhdr`로 정의하고, 같은 파일에 헤더 길이에 대한 상수도 함께 정의한다. 유저 모드 프로그램도 `<linux/if_ether.h>`를 include해 같은 정의를 쓴다. v6.18 리눅스 커널의 이더넷 헤더 정의는 인용 4와 같다.

<!-- source-ref id="if_ether" -->
```c
/*
 *	IEEE 802.3 Ethernet magic constants.  The frame sizes omit the preamble /* [!hl] */
 *	and FCS/CRC (frame check sequence). /* [!hl] */
 */

#define ETH_ALEN	6		/* Octets in one ethernet addr	 */
#define ETH_TLEN	2		/* Octets in ethernet type field */
#define ETH_HLEN	14		/* Total octets in header.	 */ /* [!hl] */
#define ETH_ZLEN	60		/* Min. octets in frame sans FCS */
#define ETH_DATA_LEN	1500		/* Max. octets in payload	 */
#define ETH_FRAME_LEN	1514		/* Max. octets in frame sans FCS */
#define ETH_FCS_LEN	4		/* Octets in the FCS		 */

/* ... */

#define ETH_P_802_3_MIN	0x0600		/* If the value in the ethernet type is more than this value
					 * then the frame is Ethernet II. Else it is 802.3 */

/* ... */

/*
 *	This is an Ethernet frame header. /* [!hl] */
 */

/* ... */

struct ethhdr {
	unsigned char	h_dest[ETH_ALEN];	/* destination eth addr	*/
	unsigned char	h_source[ETH_ALEN];	/* source ether addr	*/
	__be16		h_proto;		/* packet type ID field	*/
} __attribute__((packed));
```

`struct ethhdr`는 `__attribute__((packed))`로 정의되어 필드 사이에 패딩이 없고(당연하겠지만 없어야만 한다), 크기가 `ETH_HLEN`과 같은 14바이트이다. 받은 프레임의 바이트는 물리 장비에서 보내고 받은 순서 그대로 버퍼에 담기며, packet 소켓도 이 데이터를 바꾸지 않고 유저 모드 프로그램에 넘긴다([`man/man7/packet.7:57`](https://git.kernel.org/pub/scm/docs/man-pages/man-pages.git/tree/man/man7/packet.7?id=1bffe43b9ff4a4d8f73d33f6bd24743f4c1254d6#n57))[^13]. 엔디언은 이렇게 담긴 여러 바이트를 정수 하나로 읽을 때만 생기는 문제이다. `h_dest`와 `h_source`는 정수가 아닌 바이트 배열이므로 엔디언이 없고, 그래서 `__be` 표시도 없다. 반면 `h_proto`는 두 바이트를 16비트 정수 하나로 읽는 필드이고, 리눅스는 이 필드를 `__be16`, 즉 상위 바이트가 먼저 오는 빅 엔디언 16비트 값으로 정의한다(인용 4). 따라서 CPU의 바이트 순서와 상관없이 `ntohs()`로 읽어야 하며, `__be16`은 sparse 검사에만 쓰이는 표시라서 저장 형식은 `__u16`과 같다([`include/uapi/linux/types.h:37`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/uapi/linux/types.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n37); [`Documentation/dev-tools/sparse.rst:51`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Documentation/dev-tools/sparse.rst?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n51))[^2].

glibc의 `<net/ethernet.h>`에도 같은 방식으로 `struct ether_header`를 정의했다[^11]. `ether_type`은 `__be16` 표시 없이 `uint16_t`로 두지만 담기는 바이트는 `h_proto`와 같으므로, 이 값도 `ntohs()`로 읽어야 한다. glibc 2.44의 정의는 인용 5와 같다.

<!-- source-ref project="glibc" repo="https://sourceware.org/git/?p=glibc.git" path="sysdeps/unix/sysv/linux/net/ethernet.h" version="2.44" commit="c3a3a9808ad3ab4a3336836833f83288b672ccbf" start="38" -->
```c
/* 10Mb/s ethernet header */
struct ether_header
{
  uint8_t  ether_dhost[ETH_ALEN];	/* destination eth addr	*/
  uint8_t  ether_shost[ETH_ALEN];	/* source ether addr	*/
  uint16_t ether_type;		        /* packet type ID field	*/
} __attribute__ ((__packed__));
```

### Destination MAC address

Destination MAC address는 6바이트로, 프레임을 받을 장치의 MAC 주소이다. 이때, 맨 첫 바이트의 하위 두 비트(즉, 첫 7~8번째 비트)는 주소의 종류를 나타낸다. 최하위 비트(0x01)는 I/G(Individual/Group) 비트로, 0이면 장치 하나를 가리키는 개별 주소이고 1이면 여러 장치를 함께 가리키는 그룹 주소이다. 그 위 비트(0x02)는 U/L(Universal/Local) 비트로, 0이면 IEEE가 할당한 OUI로 시작하는 전역 주소이고 1이면 관리자나 소프트웨어가 정한 로컬 주소이다[^14]. 주소는 첫 바이트(octet 0)부터 보내고[^14], 802.3 링크는 바이트마다 최하위 비트부터 보내므로([RFC 9542 1.1절](https://www.rfc-editor.org/rfc/rfc9542#section-1.1))[^5], I/G 비트는 해당 비트열에서 가장 먼저 실제 물리적 선으로 나가는 값이다. 주소의 구성과 첫 바이트의 비트는 다이어그램 6과 같다.

```canvas
{"label": "A 48-bit MAC address from an MA-L block: the first three octets are the OUI and the last three the extension its holder assigns; in octet 0, bit 0 is the I/G bit and bit 1 the U/L bit",
 "items": [
  {"row": "addr", "x": 0, "y": 60, "h": 52, "cells": [
   {"w": 100, "label": "octet 0"},
   {"w": 100, "label": "octet 1"},
   {"w": 100, "label": "octet 2"},
   {"w": 100, "label": "octet 3"},
   {"w": 100, "label": "octet 4"},
   {"w": 100, "label": "octet 5"}
  ]},
  {"line": ["addr|0.top", [0, 16]]},
  {"line": ["addr|3.top", [300, 16]]},
  {"line": ["addr|6.top", [600, 16]]},
  {"dimension": [[0, 24], [300, 24]], "label": "OUI (24 bits)"},
  {"dimension": [[300, 24], [600, 24]], "label": "extension (24 bits)"},
  {"row": "bits", "x": 0, "y": 200, "h": 52, "cells": [
   {"w": 200, "label": "bits 7-2"},
   {"w": 100, "label": ["U/L", "bit 1"], "fill": "purple"},
   {"w": 100, "label": ["I/G", "bit 0"], "fill": "purple"}
  ]},
  {"line": ["addr|0.bottom", "bits|0.top"], "dash": true},
  {"line": ["addr|1.bottom", "bits|3.top"], "dash": true}
 ]}
```

OUI는 IEEE RA(Registration Authority)가 조직에 할당하는 24비트 값이며, 할당한 값은 MA-L 공개 목록에 올라 있다[^14][^15]. 첫 옥텟의 하위 두 비트도 OUI의 일부이고, 할당된 OUI에서는 두 비트가 모두 0이므로 OUI로 시작하는 주소는 개별 주소이자 전역 주소가 된다[^14]. 주소의 뒤 3바이트인 extension은 OUI를 받은 조직이 자기 블록 안에서 인터페이스마다 다르게 정하는 값이며, OUI와 합친 48비트 주소가 하드웨어 인터페이스 하나를 전역에서 유일하게 가리키게 한다[^14]. 비유하면 OUI는 제조사를, extension은 그 제조사가 개별 장비마다 매기는 일련번호를 가리키는 셈이며, 이 비유는 U/L 비트가 0인 전역 주소에만 맞는다. 다만 번호는 제품 하나가 아니라 인터페이스 하나마다 붙어 포트가 여럿인 NIC는 주소도 여럿이고, IEEE RA는 extension의 할당을 통제하지 않으므로 번호를 매기는 방식과 순서, 유일성 관리는 모두 조직의 몫이다[^14]. 한 조직이 OUI를 여러 개 받기도 해서, 같은 목록에서 Cisco Systems는 1,000개가 넘고 Intel은 600개가 넘는다[^15]. 유선 NIC과 무선 AP로 잘 알려진 제조사의 OUI를 하나씩 들면 표 1과 같다.

| OUI | 등록 조직 |
|---|---|
| `00:00:0C` | Cisco Systems, Inc |
| `00:09:5B` | NETGEAR |
| `00:0C:E7` | MediaTek Inc |
| `00:10:18` | Broadcom |
| `00:1B:21` | Intel Corporate |
| `00:A0:C6` | Qualcomm Inc. |
| `00:E0:4C` | REALTEK SEMICONDUCTOR CORP. |
| `24:A4:3C` | Ubiquiti Inc |
| `50:C7:BF` | TP-LINK TECHNOLOGIES CO.,LTD. |

다이어그램 6과 표 1은 IEEE RA가 OUI와 주소 $2^{24}$개를 함께 할당하는 MA-L(Large) 블록의 경우이다[^14]. IEEE RA는 이보다 작은 MA-M(Medium)과 MA-S(Small) 블록도 할당한다. 이때 IEEE RA는 자신에게 할당된 OUI 아래에서 MA-M은 앞 28비트, MA-S는 앞 36비트를 조직에 할당하므로, 조직이 정하는 extension은 각각 20비트와 12비트이다[^14]. 따라서 이런 주소는 앞 3바이트만으로는 조직을 알 수 없고, 앞 28비트나 36비트를 MA-M이나 MA-S 공개 목록에서 찾아야 한다[^14].

리눅스에서는 `is_multicast_ether_addr()`([`include/linux/etherdevice.h:130`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/linux/etherdevice.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n130))가 I/G 비트를, `is_local_ether_addr()`([`include/linux/etherdevice.h:163`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/linux/etherdevice.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n163))가 U/L 비트를 검사한다[^2]. `is_multicast_ether_addr()`는 주소의 앞 바이트들을 정수 하나로 읽기 때문에, CPU가 빅 엔디언인지에 따라 첫 바이트의 위치를 다르게 계산한다. v6.18의 두 함수는 인용 6과 같다.

<!-- source-ref id="etherdevice" -->
```c
/**
 * is_multicast_ether_addr - Determine if the Ethernet address is a multicast.
 * @addr: Pointer to a six-byte array containing the Ethernet address
 *
 * Return: true if the address is a multicast address.
 * By definition the broadcast address is also a multicast address.
 */
static inline bool is_multicast_ether_addr(const u8 *addr)
{
#if defined(CONFIG_HAVE_EFFICIENT_UNALIGNED_ACCESS)
	u32 a = *(const u32 *)addr;
#else
	u16 a = *(const u16 *)addr;
#endif
#ifdef __BIG_ENDIAN
	return 0x01 & (a >> ((sizeof(a) * 8) - 8)); /* [!hl] */
#else
	return 0x01 & a; /* [!hl] */
#endif
}

/* ... */

/**
 * is_local_ether_addr - Determine if the Ethernet address is locally-assigned one (IEEE 802).
 * @addr: Pointer to a six-byte array containing the Ethernet address
 *
 * Return: true if the address is a local address.
 */
static inline bool is_local_ether_addr(const u8 *addr)
{
	return 0x02 & addr[0]; /* [!hl] */
}
```

그룹 주소 가운데 일부는 용도가 정해져 있으며, 잘 알려진 주소와 대역은 표 2와 같다.

| 주소(대역) | 용도 |
|---|---|
| `FF:FF:FF:FF:FF:FF` | 브로드캐스트 |
| `01:00:5E:00:00:00`~`01:00:5E:7F:FF:FF` | IPv4 멀티캐스트 |
| `33:33:00:00:00:00`~`33:33:FF:FF:FF:FF` | IPv6 멀티캐스트 |

브로드캐스트 주소는 모든 비트가 1인 주소이므로 I/G 비트도 1이다[^12]. ARP는 IPv4 주소에 대응하는 MAC 주소를 묻는 요청을 이 주소로 보내, 같은 Ethernet의 모든 장치가 받게 한다[^16]. IPv4 멀티캐스트 주소는 `01:00:5E:00:00:00`의 하위 23비트에 IPv4 그룹 주소의 하위 23비트를 넣어 만든다([RFC 1112 6.4절](https://www.rfc-editor.org/rfc/rfc1112#section-6.4))[^17]. 그래서 넷째 바이트의 최상위 비트는 늘 0이고, 대역은 `01:00:5E:7F:FF:FF`에서 끝난다. IPv6 멀티캐스트 주소는 `33:33` 뒤에 IPv6 그룹 주소의 마지막 4바이트를 넣는다([RFC 2464 7절](https://www.rfc-editor.org/rfc/rfc2464#section-7))[^18].

이 주소들은 모두 I/G 비트가 1인 그룹 주소이므로, 장치 하나의 고유 주소로 쓸 수 없다. 리눅스에서 Ethernet 장치의 주소를 바꾸는 `eth_prepare_mac_addr_change()`([`net/ethernet/eth.c:272`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/net/ethernet/eth.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n272))도 `is_valid_ether_addr()`([`include/linux/etherdevice.h:205`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/linux/etherdevice.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n205))로 그룹 주소와 모든 비트가 0인 주소를 걸러 내고, 이런 주소에는 `-EADDRNOTAVAIL`을 돌려준다[^2].

### Source MAC address

Source MAC address는 6바이트로, 프레임을 보낸 장치의 MAC 주소이다. 주소의 구성은 Destination MAC address와 같다. 리눅스 브리지([`net/bridge/br_input.c:347`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/net/bridge/br_input.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n347), `br_handle_frame()`)는 Source MAC address가 그룹 주소이거나 모든 비트가 0인 프레임은 받지 않고 버린다[^2].

### EtherType

EtherType은 2바이트로, 헤더 뒤의 데이터에 담긴 프로토콜을 식별한다([RFC 9542 3절](https://www.rfc-editor.org/rfc/rfc9542#section-3))[^5]. Ethernet은 L2 프로토콜이므로, 그 프로토콜은 IPv4·IPv6 같은 L3 프로토콜일 수도 있고 LLDP처럼 같은 L2에서 동작하는 프로토콜일 수도 있다. IEEE 802.3은 이 필드를 Length/Type 필드라고 부르고[^19], MAC 의사코드에서는 길이나 타입을 담는 `lengthOrTypeField`로 둔다[^4]. 값이 1500(0x05DC) 이하이면 데이터 필드의 길이이고, 1536(0x0600) 이상이면 EtherType이다([RFC 9542 3절](https://www.rfc-editor.org/rfc/rfc9542#section-3))[^5]. 리눅스도 `ETH_P_802_3_MIN`(0x0600)을 경계로 두 경우를 나눈다(인용 4).

값이 길이이면 데이터 필드는 LLC(Logical Link Control) 헤더로 시작하고, 프로토콜은 그 안의 DSAP와 SSAP가 나타낸다. DSAP와 SSAP는 각각 받는 쪽과 보내는 쪽 프로토콜을 가리키는 8비트 식별자이고, 그 뒤에 1바이트나 2바이트의 Control 필드가 온다([RFC 9542 3절](https://www.rfc-editor.org/rfc/rfc9542#section-3); [`include/net/llc_pdu.h:18`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/net/llc_pdu.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n18), `LLC_PDU_LEN_I`, `LLC_PDU_LEN_U`)[^5][^2]. 리눅스의 `eth_type_trans()`([`net/ethernet/eth.c:155`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/net/ethernet/eth.c?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n155))는 길이로 해석되는 LLC 프레임을 `ETH_P_802_2`로 분류하고, 값이 EtherType이면 그 값을 그대로 프레임의 프로토콜 ID로 돌려준다[^2]. 잘 알려진 EtherType 값과 리눅스 상수([`include/uapi/linux/if_ether.h:44`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/uapi/linux/if_ether.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n44), `ETH_P_*`)는 표 3과 같다[^20][^21][^2].

| EtherType | 프로토콜 | 정의 | Linux 심볼명 |
|---|---|---|---|
| `0x0800` | IPv4 | RFC 894 | `ETH_P_IP` |
| `0x0806` | ARP | RFC 826 | `ETH_P_ARP` |
| `0x8100` | VLAN 태그 (C-TAG) | IEEE 802.1Q | `ETH_P_8021Q` |
| `0x86DD` | IPv6 | RFC 2464 | `ETH_P_IPV6` |
| `0x8809` | Slow Protocols | IEEE 802.3 | `ETH_P_SLOW` |
| `0x8847` | MPLS | RFC 3032 | `ETH_P_MPLS_UC` |
| `0x8864` | PPPoE 세션 | RFC 2516 | `ETH_P_PPP_SES` |
| `0x888E` | EAPOL (802.1X) | IEEE 802.1X | `ETH_P_PAE` |
| `0x88A8` | 서비스 VLAN 태그 (S-TAG, 802.1ad) | IEEE 802.1Q | `ETH_P_8021AD` |
| `0x88CC` | LLDP | IEEE 802.1AB | `ETH_P_LLDP` |

`0x8100`과 `0x88A8`은 EtherType 자리에 VLAN 태그가 들어왔다는 표시이다[^20]. 태그는 4바이트이며, 앞 2바이트는 태그의 종류를 나타내는 이 EtherType 값이고, 뒤 2바이트 TCI는 우선순위 PCP 3비트, DEI 1비트, VLAN 번호 VID 12비트로 나뉜다([`include/linux/if_vlan.h:73`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/linux/if_vlan.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n73), `VLAN_PRIO_MASK`, `VLAN_CFI_MASK`, `VLAN_VID_MASK`)[^2]. 데이터의 프로토콜을 나타내는 EtherType은 태그 다음에 오므로, 태그가 하나 붙은 프레임의 헤더는 18바이트이다([RFC 9542 3절](https://www.rfc-editor.org/rfc/rfc9542#section-3); [`include/linux/if_vlan.h:19`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/include/linux/if_vlan.h?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18#n19), `VLAN_ETH_HLEN`)[^5][^2]. 태그가 붙은 프레임의 최대 길이도 태그 4바이트만큼 늘어난 1522바이트이다[^10]. 태그의 비트 구성은 다이어그램 7과 같다.

```memory-table
{"label": "A 4-byte 802.1Q tag in two 16-bit rows: an EtherType value (0x8100 or 0x88A8) that names the tag, then the TCI with a 3-bit PCP, a 1-bit DEI and a 12-bit VID", "unit": "bit", "cols": 16, "order": "desc", "fields": [["EtherType", 16, "purple"], ["VID", 12, "purple"], ["DEI", 1, "purple"], ["PCP", 3, "purple"]]}
```

## References

[^1]: IEEE, "[IEEE Standard for Ethernet](https://doi.org/10.1109/IEEESTD.2022.9844436)," IEEE Std 802.3-2022.
[^2]: Linux v6.18.60, commit `f45ed76d6791`, [`stable/linux.git`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/?id=f45ed76d6791cf72ff1265ee93511175bbbc2f18).
[^3]: S. Muller, "[802.3 Revision Request 1067](https://www.ieee802.org/3/maint/requests/maint_1067.pdf)," IEEE 802.3 Working Group, Sep. 2000.
[^4]: IEEE 802.3z Task Force, "[P802.3z/D0, Clause 4 MAC pseudocode (4.2.7–4.2.9)](https://www.ieee802.org/3/z/public/presentations/nov1996/MMbstpas.pdf)," Nov. 1996.
[^5]: D. Eastlake 3rd, J. Abley, and Y. Li, "[IANA Considerations and IETF Protocol and Documentation Usage for IEEE 802 Parameters](https://www.rfc-editor.org/rfc/rfc9542)," RFC 9542, 2024.
[^6]: UNH InterOperability Laboratory, "[10BASE-T Medium Attachment Unit](https://www.iol.unh.edu/sites/default/files/knowledgebase/ethernet/10basetmau.pdf)," University of New Hampshire.
[^7]: C. Mick, C. DiMinico, S. Raghavan, S. Rao, and M. Hatamian, "[802.3ab: A Tutorial Presentation](https://grouper.ieee.org/groups/802/3/ab/pub/tutorial.pdf)," IEEE 802.3ab Task Force, Mar. 1998.
[^8]: UNH InterOperability Laboratory, "[Clause 4 Media Access Control (MAC) Test Suite](https://www.iol.unh.edu/sites/default/files/testsuites/ethernet/MAC/MAC_Test_Suite_v5.0.pdf)," Version 5.0, University of New Hampshire, Jan. 2011.
[^9]: N. Finn, "[Suggested changes to Clause 12, IEEE P802.1AC-REV/D2.1](https://grouper.ieee.org/groups/802/1/files/public/docs2014/acrev-nfinn-comment33-0914-v01.pdf)," IEEE 802.1 Working Group, Sep. 2014.
[^10]: IEEE Standards Association, "[IEEE Standards Interpretation for IEEE Std 802.1D-1999, IEEE Std 802.1Q-1998 and IEEE Std 802.3, 2000 Edition](https://standards.ieee.org/wp-content/uploads/import/documents/interpretations/802.1D_802.1Q_802.3_interp.pdf)," Jul. 2002.
[^11]: GNU C Library 2.44, commit `c3a3a9808ad3`, [`sysdeps/unix/sysv/linux/net/ethernet.h`](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/unix/sysv/linux/net/ethernet.h;hb=c3a3a9808ad3ab4a3336836833f83288b672ccbf).
[^12]: C. Hornig, "[A Standard for the Transmission of IP Datagrams over Ethernet Networks](https://www.rfc-editor.org/rfc/rfc894)," RFC 894, 1984.
[^13]: Linux man-pages 6.20, commit `1bffe43b9ff4`, [`man-pages.git`](https://git.kernel.org/pub/scm/docs/man-pages/man-pages.git/tree/?id=1bffe43b9ff4a4d8f73d33f6bd24743f4c1254d6).
[^14]: IEEE Registration Authority, "[Guidelines for Use of Extended Unique Identifier (EUI), Organizationally Unique Identifier (OUI), and Company ID (CID)](https://standards.ieee.org/wp-content/uploads/import/documents/tutorials/eui.pdf)," 2017.
[^15]: IEEE Registration Authority, "[MA-L public listing](https://standards-oui.ieee.org/oui/oui.txt)."
[^16]: D. C. Plummer, "[An Ethernet Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826)," RFC 826, 1982.
[^17]: S. Deering, "[Host Extensions for IP Multicasting](https://www.rfc-editor.org/rfc/rfc1112)," RFC 1112, 1989.
[^18]: M. Crawford, "[Transmission of IPv6 Packets over Ethernet Networks](https://www.rfc-editor.org/rfc/rfc2464)," RFC 2464, 1998.
[^19]: T. Jeffree, "[802.3 Revision Request 1095](https://www.ieee802.org/3/maint/requests/maint_1095.pdf)," IEEE 802.3 Working Group, Jun. 2002.
[^20]: IEEE Registration Authority, "[EtherType public listing](https://standards-oui.ieee.org/ethertype/eth.txt)."
[^21]: IANA, "[IEEE 802 Numbers](https://www.iana.org/assignments/ieee-802-numbers/)," 2024.
