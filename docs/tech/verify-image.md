# Verify an image downloaded from https://raspi.debian.net/

## Verify the download hash

The hash stored in the corresponding metadata file is not in the same format as the output from `sha512sum` and cannot be directly compared. The output can be converted to a comparable format (base64 digest) using the following command:

```bash
image_file_name=[compressed image file]
openssl dgst -sha512 -binary "$image_file_name"| base64 | tr -d '\n' | head -c -2; echo
```

The result of this command can then be compared to the hash (`cloud.debian.org/digest`) stored in the metadata.
