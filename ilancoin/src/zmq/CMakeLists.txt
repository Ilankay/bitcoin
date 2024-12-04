# Copyright (c) 2023-present The Ilancoin Core developers
# Distributed under the MIT software license, see the accompanying
# file COPYING or https://opensource.org/license/mit/.

add_library(ilancoin_zmq STATIC EXCLUDE_FROM_ALL
  zmqabstractnotifier.cpp
  zmqnotificationinterface.cpp
  zmqpublishnotifier.cpp
  zmqrpc.cpp
  zmqutil.cpp
)
target_compile_definitions(ilancoin_zmq
  INTERFACE
    ENABLE_ZMQ=1
)
target_link_libraries(ilancoin_zmq
  PRIVATE
    core_interface
    univalue
    zeromq
)
