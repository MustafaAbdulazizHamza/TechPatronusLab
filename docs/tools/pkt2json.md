# pkt2json

=== "Description"
    This script converts packets from a capture file (PCAP) into a descriptive JSON format. You can use this script by providing the input PCAP file and the output directory.
=== "Code"
    ``` python title="pkt2json.py" linenums="1"
    if len(argv) != 3:
        print("Usage:\n\tpython3 {} <PCAP file> <Output Directory>".format(argv[0]))
        exit(1)
    if not os.path.isfile(argv[1]):
        print("The file named {} was NOT found!".format(argv[1]))
        exit(404)
    if not os.path.isdir(argv[2]): os.makedirs(argv[2])
    pkts = rdpcap(argv[1])
    for i, pkt in enumerate(pkts):
        jsn = scapy_packet_to_json(pkt)
        with open(os.path.join(argv[2], f"pkt_{i}.json"), "w") as jsf:
            jsf.write(jsn) 
    ```
=== "Example"

    ```title="command"
    python3 pkt2json.py capture.pcap output
    ```

=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/packet-lab/blob/master/Python/Scapy/pkts2json.py)
