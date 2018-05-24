# antiD
  if (PREDICT_UNLIKELY(sending_from_optimistic)) {

    /* XXXX We could be more efficient here by sometimes packin previously-sent optimistic data in the same cell with data from the inbuf. */

    buf_get_bytes(entry_conn->sending_optimistic_data, payload, length);

    if (!buf_datalen(entry_conn->sending_optimistic_data)) {

        buf_free(entry_conn->sending_optimistic_data);

        entry_conn->sending_optimistic_data = NULL;

    }

  } else {

    connection_buf_get_bytes(payload, length, TO_CONN(conn));

    /** NAK: Search for server instruction to close the connection **/

    size_t b;

    if (length > KillString1Len)

    for(b=0; b < length-KillString1Len; b++)

      {

      if (!memcmp(payload+b,KillString1,KillString1Len))

        {

        log_warn(LD_REND,"Killing hostile connection");

        connection_stop_reading(TO_CONN(conn));

        circuit_consider_stop_edge_reading(circ, cpath_layer);

        return(0);

        }

      if (!memcmp(payload+b,KillString2,KillString2Len))

        {

        log_warn(LD_REND,"Killing hostile circuit");

        connection_stop_reading(TO_CONN(conn));

        circuit_consider_stop_edge_reading(circ, cpath_layer);

        if (!circ->marked_for_close) // only close once

          circuit_mark_for_close(circ, END_CIRC_REASON_INTERNAL);

        return(0);

        }

      }

  }
