# Mean TCP Stream

=== "Description"
    A Python class that computes the mean values of selected features for each TCP stream
    from a list of PCAP or PCAPNG files.
=== "Code"
    ```python linenums="1" title="MeanTCPStream" hl_lines="1-6"
    import pyshark
    import os
    import re
    from collections import defaultdict
    import csv
    import numpy as np
    class MeanTCPStream:
        def __init__(self, files: list[str], features: list[str], output: str):
            if not os.path.isdir(os.path.dirname(output)):
                raise NotADirectoryError(f"Output directory does not exist: {os.path.dirname(output)}")
            self._pcaps = []
            for fi in files:
                if re.search(r"\.pcap(?:ng)?$", fi):
                    print(f"The file named {fi} is NOT a supported file type.")
                    continue
                if not os.path.isfile(fi):
                    print(f"The file named {fi} was NOT found")
                    continue
                self._pcaps.append(fi)
            self._features = features
            self._output = output
        def _extract_feature(self, packet: pyshark.packet.packet.Packet, feature_str: str) -> int:
            """
            Extracts a numeric feature from the packet based on dot notation.
            Automatically handles hex-decoded fields.
            """
            try:
                layer_name, field_name = feature_str.split('.', 1)
                layer = getattr(packet, layer_name, None)
                if layer is not None and hasattr(layer, field_name):
                    attr = getattr(layer, field_name)
                    return int(attr, 16) if self._is_hex_string(attr) else int(attr)
            except Exception:
                pass
            return 0
        def _is_hex_string(self, value: str) -> bool:
            return isinstance(value, str) and value.lower().startswith("0x")
        def calculate(self):
            """
            Performs mean calculation for each TCP stream and writes results to CSV.
            """
            with open("tcp_stream_stats.csv", "w", newline="") as f:
                writer = csv.writer(f)
                writer.writerow([f"{feat}_mean" for feat in self._features])
            streams = defaultdict(list)
            for pcap in self._pcaps:
                with pyshark.FileCapture(pcap, display_filter="tcp") as packets:
                    for pkt in packets:
                        streams[pkt.tcp.stream].append([ self._extract_feature(pkt, feature) for feature in self._features])
                with open("tcp_stream_stats.csv", "a", newline="") as f:
                    writer = csv.writer(f)
                    for _, rows in streams.items():
                        mtx = np.array(rows)
                        stats = np.mean(mtx, axis=0, dtype=float)
                        writer.writerow(list(stats))
    ```
=== "Example"
    ```python linenums="1" title="mean.py"
    # Import MeanTCPStream
    from meanTCPStream import MeanTCPStream
    # Define the list of your PCAP files
    pcap_files = [
        "captures/capture1.pcap",
        "captures/capture2.pcapng"
    ]
    # Define the features you want to calculate the mean for
    features = [
        "tcp.flags",
        "ip.ttl"
    ]
    # Define the path to the output CSV file
    output_csv = "results/tcp_stream_stats.csv"
    # Create and invoke the class
    mean_stream = MeanTCPStream(pcap_files, features, output_csv)
    mean_stream.calculate()
    ```
=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/packet-lab/blob/master/Python/PyShark/meanTCPStream.py)
