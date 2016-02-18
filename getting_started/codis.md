# Codis
codis-admin --dashboard=192.168.88.6:18080 --group-add   --gid=1 --addr=192.168.88.6:56379
codis-admin --dashboard=192.168.88.6:18081 --group-add   --gid=1 --addr=192.168.88.6:56380
codis-admin --dashboard=192.168.88.6:18082 --group-add   --gid=1 --addr=192.168.88.6:56381
codis-admin --dashboard=192.168.88.6:18080 --slot-action --create-range --beg=0 --end=1023 --gid=1
codis-admin --dashboard=192.168.88.6:18081 --slot-action --create-range --beg=0 --end=1023 --gid=1
codis-admin --dashboard=192.168.88.6:18082 --slot-action --create-range --beg=0 --end=1023 --gid=1
codis-admin  -v --dashboard=192.168.88.6:18080   --create-proxy  --addr=192.168.88.6:11080
codis-admin  --dashboard=192.168.88.6:18081   --create-proxy  --addr=192.168.88.6:11081
codis-admin  --dashboard=192.168.88.6:18082   --create-proxy  --addr=192.168.88.6:11082

