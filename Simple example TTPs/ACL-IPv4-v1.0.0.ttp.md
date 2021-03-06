```json
{
  "NDM_metadata": {
    "authority": "org.opennetworking.fawg",
    "type": "TTPv1",
    "name": "ACL-IPv4",
    "version": "1.0.0",
    "OF_protocol_version": "1.3.3",
    "doc": ["TTP supporting ACLs and IPv4 unicast and multicast"]
  },

  "security": {
    "doc": ["This NDM is not published for use. It is an example for",
            "illustrative purposes only."]
  },
  
  "table_map": {
    "ACL": 0,
    "IPv4": 1
  },

  "identifiers": [
    {"var": "<Router_MAC_DA>",
     "doc": "A unicast MAC address on a Router port."},
    {"var": "<Router_IP>",
     "doc": ["An IP address used to reach L3 control functions,",
             "e.g. a loopback address in the Router."]},
    {"var": "<LocalSubnet>",
     "doc": "An IP subnet (address prefix) allocated to a directly attached L2 network or link."},
    {"var": "<port_no>",
     "doc": "A valid port number on the logical switch."},
    {"var": "<local_MAC>",
     "doc": "The unicast MAC address of a Router port on which a new L2 frame is transmitted."},
    {"var": "<dest_MAC>",
     "doc": "The destination MAC address for a new L2 frame."},
    {"var": "<<group_entry_types:name>>",
     "doc": ["An OpenFlow group identifier (integer) identifying a group table entry",
             "of the type indicated by the variable name"]}
  ],
  
  "meter_table": {
    "meter_types": [
      {"name": "ControllerMeterType",
       "bands": [{"type": "DROP", "rate": "1000..10000", "burst": "50..200"}]
      }
    ],
    "built_in_meters": [
      {"name": "ControllerMeter", "meter_id": 1,
        "type": "ControllerMeterType", "bands": [{"rate": 2000, "burst": 75}]}
    ]
  },
  
  "flow_tables": [
    {
      "name": "ACL",
      "doc": "Simple 5-tuple firewalling ACL table.",
      "flow_mod_types": [
        {
          "name": "IP5-Tuple-Block",
          "doc": ["This type allows matching on an IP 5-tuple and",
                  "dropping packets."],
          "match_set": [
            {"exactly_one": [
              [
                {"field": "IPV4_SRC", "match_type": "mask"}, 
                {"field": "IPV4_DST", "match_type": "mask"}, 
                {"field": "TCP_SRC",  "match_type": "mask"}, 
                {"field": "TCP_DST",  "match_type": "mask"}
              ],
              [
                {"field": "IPV4_SRC", "match_type": "mask"}, 
                {"field": "IPV4_DST", "match_type": "mask"}, 
                {"field": "UDP_SRC",  "match_type": "mask"}, 
                {"field": "UDP_DST",  "match_type": "mask"}
              ]
            ]}
          ],
          "instruction_set": [
            {"instruction": "CLEAR_ACTIONS"}
          ]
        },
        {
          "name": "IP-5Tuple-Intercept",
          "doc": ["This type allows matching on an IP 5-tuple and",
                  "forwarding to the controller."],
          "match_set": [
            {"exactly_one": [
              [
                {"field": "IPV4_SRC", "match_type": "mask"}, 
                {"field": "IPV4_DST", "match_type": "mask"}, 
                {"field": "TCP_SRC",  "match_type": "mask"}, 
                {"field": "TCP_DST",  "match_type": "mask"}
              ],
              [
                {"field": "IPV4_SRC", "match_type": "mask"}, 
                {"field": "IPV4_DST", "match_type": "mask"}, 
                {"field": "UDP_SRC",  "match_type": "mask"}, 
                {"field": "UDP_DST",  "match_type": "mask"}
              ]
            ]}
          ],
          "instruction_set": [
            {"zero_or_one":
              {"instruction": "METER", "meter_name": "ControllerMeter",
               "doc": ["This meter may be used to limit the rate of",
                       "PACKET_IN frames sent to the controller"]}
            },
            {"intruction": "APPLY_ACTIONS",
              "actions": [
                {"action": "OUTPUT", "port": "CONTROLLER"}]
            }
          ]
        },
        {
          "name": "IP-5Tuple-Allow",
          "doc": ["This type allows matching on an IP 5-tuple and",
                  "sending on to the L2 table, overriding a lower",
                  "priority block or intercept."],
          "match_set": [
            {"exactly_one": [
              [
                {"field": "IPV4_SRC", "match_type": "mask"}, 
                {"field": "IPV4_DST", "match_type": "mask"}, 
                {"field": "TCP_SRC",  "match_type": "mask"}, 
                {"field": "TCP_DST",  "match_type": "mask"}
              ],
              [
                {"field": "IPV4_SRC", "match_type": "mask"}, 
                {"field": "IPV4_DST", "match_type": "mask"}, 
                {"field": "UDP_SRC",  "match_type": "mask"}, 
                {"field": "UDP_DST",  "match_type": "mask"}
              ]
            ]}
          ],
          "instruction_set": [
              {"instruction": "GOTO_TABLE", "table": "IPv4"}
          ]
        },
        {
          "name": "IPv4-Route",
          "priority": 1,
          "doc": "Direct IPv4 packets matching Router MAC address to IPv4 flow table.",
          "match_set": [
            {"field": "ETH_TYPE", "value": "0x800"},
            {"field": "ETH_DST", "value": "<Router_MAC_DA>"}
          ],
          "instruction_set": [
            {"instruction": "GOTO_TABLE", "table": "IPv4"}
          ]
        }
      ],
      "built_in_flow_mods": [
        {
          "name": "Router-ARP",
          "priority": 2,
          "doc": "Send ARP packets targeted to Router to controller.",
          "match_set": [
            {"field": "ARP_TPA", "value": "<Router_IP>"}
          ],
          "instruction_set": [
            {"intruction": "APPLY_ACTIONS",
              "actions": [
                {"action": "OUTPUT", "port": "CONTROLLER"}]
            }
          ]
        }
      ]
    },
    {
      "name": "IPv4",
      "doc": ["IPv4 forwarding table.  To achieve LPM the flow_mod",
              "priority must be the length of the prefix mask.",
              "This is indicated by using the 'prefix' match_type."],
      "flow_mod_types": [
        {
          "name": "Unicast",
          "doc": ["LPM forwarding entry. Valid only if the priority value",
                  "matches the length of the prefix mask."],
          "match_set": [
            {"field": "IPV4_DST", "match_type": "prefix"}
          ],
          "instruction_set": [
            {"intruction": "APPLY_ACTIONS",
              "actions": [
                {"action": "GROUP", "group_id": "<NextHop>"}]
            }
          ]
        },
        {
          "name": "Local-ARP",
          "doc": ["Match for local subnet address needing ARP."],
          "match_set": [
            {"field": "IPV4_DST", "value": "<LocalSubnet>", "match_type": "prefix"}
          ],
          "instruction_set": [
            {"instruction": "APPLY_ACTIONS",
              "actions": [
                {"action": "OUTPUT", "port": "CONTROLLER"}]
            }
          ]
        },
        {
          "name": "Multicast",
          "doc": ["Exact match forwarding entry."],
          "match_set": [
            {"field": "IPV4_DST"}
          ],
          "instruction_set": [
            {"instruction": "APPLY_ACTIONS",
              "actions": [
                {"action": "GROUP", "group_id": "<Multicast>"}]
            }
          ]
        },
        {
          "name": "Default-Route",
          "doc": ["Use to create a default routing entry."],
          "priority": 0,
          "match_set": [],
          "instruction_set": [
            {"instruction": "APPLY_ACTIONS",
              "actions": [
                {"action": "GROUP", "group_id": "<NextHop>"}]
            }
          ]
        }
      ]
    }
  ],

  "group_entry_types": [
    {
      "name": "NextHop",
      "doc": ["Decrement IP TTL and add L2 header for next hop.",
              "Entry per next hop IP address."],
      "group_type": "INDIRECT",
      "bucket_types": [
        {"name": "KnownMAC",
         "action_set": [
           {"action": "DEC_NW_TTL"},
           {"action": "SET_FIELD", "type": "ETH_SRC", "value": "<local_MAC>"},
           {"action": "SET_FIELD", "type": "ETH_DST", "value": "<dest_MAC>"},
           {"action": "OUTPUT", "port": "<port_no>"}]
        }
      ]
    },
    {
      "name": "Multicast",
      "doc": ["Multicast group entry.",
              "Entry per multicast address."],
      "group_type": "ALL",
      "bucket_types": [
        {"name": "Branch",
         "action_set": [{"action": "GROUP", "group_id": "<NextHop>"}]
        }
      ]
    }
  ],

  "parameters": [
    {"name": "ACL::TableSize", "type": "integer"},
    {"name": "IPv4::TableSize", "type": "integer"}
  ]
}
```
