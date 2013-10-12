repl.h
======

Create a repl with eval/print/error hooks with given stdin, stdout, and stderr streams

## install

```sh
$ clib install jwerle/repl.h
```

## usage

The `repl` interface is more or less just an api for defining functions
that answer the "eval", and "print" parts of a REPL routine. The "read"
part is just a read from a defined "stdin" stream that passes a buffer
to the defined "eval" function which returns a `char *` that is passed
to the defined "print" function. The print function if not defined will
print to the defined "stdout" stream.

```c


#include <repl.h>

static char *
eval (repl_session_t *sess, char *buf);

static void
error (repl_session_t *sess, char *err);

 
int
main (void) {
  int rc;
  repl_session_opts opts;

  opts.prompt = "js>";
  opts.eval_cb = eval;
  opts.error_cb = error;

  repl_session_t *sess = repl_session_new(&opts);

  // main loop happens here
  rc = repl_session_start(sess);

  repl_session_destroy(sess);
  
  return rc > 0? 1 : 0;
}


static char *
eval (repl_session_t *sess, char *buf) {
  char cmd[4096];
  char *out_str;
  char tmp[2048];
  buffer_t *out = buffer_new();
  FILE *node;

  sprintf(cmd, "/usr/bin/env node -e \"%s\"", buf);
  node = popen(cmd, "r");

  if (NULL == node) {
    repl_session_set_error("Couldn't start node");
    sess->rc = 1;
    return NULL;
  }
  
  while (NULL != fgets(tmp, sizeof(tmp) - 1, node)) {
    buffer_append(out, tmp);
  }

  fclose(node);

  out_str = strdup(buffer_string(out));
  buffer_free(out);
  return out_str;
}

static void
error (repl_session_t *sess, char *err) {
  fprintf(sess->stdout, "error: %s", err);
}
```

## license

MIT
