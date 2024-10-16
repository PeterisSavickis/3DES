Praktiskais darbs kursā Lietišķā Kriptiogrāfija I

# Uzstādīšanas instrukcija

1. Lejuplādē pievienoto Jupyter Notebook failu.
2. Izmanto sev vispiemērotāko vidi kas spēj palaist ipynb failus, piemēram, Anaconda Navigator - https://www.anaconda.com/products/navigator
3. Main metodē ieraksti ceļu uz failu ko vēlies šifrēt
4. Vari modificēt šifrēto un atšifrēto failu nosaukumus.

Droši veidojiet atsevišķus zarus un papildiniet kodu. Ja vēlaties iepludināt izmaiņas Main zarā, lūdzu izveidojiet Pull Request (PR). 

Šis kods ir domāts, lai izprastu 3DES algoritmu darbības principus.





# 3DES Algoritma darbības princips

Šeit ir aprakstīts, kā darbojas failā pieejamais 3DES šifrēšanas un atšifrēšanas process, soli pa solim.

## Main sadaļa

```python
# Atslēgas (katra ir 8 baiti)
key1 = b'12345678'
key2 = b'23456789'
key3 = b'87654321'

# Šifrē PDF
encrypt_file('test.pdf', 'encrypted_sample.pdf', key1, key2, key3)

# Atšifrē PDF
decrypt_file('encrypted_sample.pdf', 'decrypted_sample.pdf', key1, key2, key3)
```

Šajā daļā tiek izmantotas trīs atslēgas, un šifrēšanai tiek izsaukta `encrypt_file` funkcija, bet atšifrēšanai `decrypt_file`.

## `encrypt_file` funkcija

```python
def encrypt_file(input_file, output_file, key1, key2, key3):
    with open(input_file, 'rb') as f:
        data = f.read()  # Nolasām faila saturu
    encrypted_data = encrypt(data, key1, key2, key3)  # Šifrējam datus
    with open(output_file, 'wb') as f:
        f.write(encrypted_data)  # Saglabājam šifrēto failu
```

1. Atver failu un nolasa tā saturu kā bitus (`data`).
2. Izsauc `encrypt` funkciju, kas šifrē datus ar trīs atslēgām.
3. Saglabā šifrētos datus jaunā failā (`output_file`).

## `encrypt` funkcija

```python
def encrypt(data, key1, key2, key3):
    # Konvertējam datus un atslēgas bitos
    data_bits = bytearray_to_bits(data)
    key1_bits = bytearray_to_bits(key1)
    key2_bits = bytearray_to_bits(key2)
    key3_bits = bytearray_to_bits(key3)

    # Datu sadalīšana pa blokiem (64 bitu blokos)
    blocks = split_into_blocks(data_bits, 64)

    encrypted_blocks = []
    for block in blocks:
        # Pirmā šifrēšana ar key1
        encrypted_block = des_encrypt(block, key1_bits)
        # Atšifrēšana ar key2 (Triple-DES daļa)
        decrypted_block = des_decrypt(encrypted_block, key2_bits)
        # Otrā šifrēšana ar key3
        encrypted_block_again = des_encrypt(decrypted_block, key3_bits)
        encrypted_blocks.append(encrypted_block_again)

    # Apvienojam šifrētos blokus
    encrypted_data_bits = [bit for block in encrypted_blocks for bit in block]
    return bits_to_bytearray(encrypted_data_bits)
```

Šī funkcija veic Triple DES (3DES) šifrēšanu šādi:
1. Dati un atslēgas tiek konvertētas bitos.
2. Dati tiek sadalīti 64 bitu blokos.
3. Katru bloku:
   - Pirmais DES šifrēšanas solis tiek veikts ar `key1`.
   - Tad bloku atšifrē ar `key2`.
   - Visbeidzot, veic otro šifrēšanu ar `key3`.

## `des_encrypt` funkcija

```python
def des_encrypt(data, key):
    data_bits = data
    key_bits = key

    # Sākotnējā permutācija
    permuted_data = initial_permutation(data_bits)

    # Sadalām divās daļās
    L, R = permuted_data[:32], permuted_data[32:]

    # DES šifrēšanas cikls (16 kārtas)
    for i in range(16):
        L, R = R, xor_bits(L, feistel(R, key_bits))

    # Apvienojam un veicam galīgo permutāciju
    combined_data = final_permutation(L + R)
    return combined_data
```

- **Sākotnējā permutācija**: Datu biti tiek pārkārtoti pēc definētas tabulas.
- **Sadalīšana divās daļās**: Dati tiek sadalīti divās 32 bitu daļās (`L` un `R`).
- **16 kārtu šifrēšanas cikls**: 16 iterācijas ar Feistel funkciju un XOR operācijām.
- **Galīgā permutācija**: Apvieno rezultātu un veic pēdējo permutāciju, lai iegūtu šifrētos datus.

## `des_decrypt` funkcija

```python
def des_decrypt(data, key):
    # Darbojas tāpat kā des_encrypt, tikai pretējā secībā
    ...
```

## `decrypt_file` funkcija

```python
def decrypt_file(input_file, output_file, key1, key2, key3):
    with open(input_file, 'rb') as f:
        data = f.read()  # Nolasām šifrēto failu
    decrypted_data = decrypt(data, key1, key2, key3)  # Atšifrējam datus
    with open(output_file, 'wb') as f:
        f.write(decrypted_data)  # Saglabājam atšifrēto failu
```

- Šī funkcija nolasīs šifrēto failu, atšifrēs to ar `decrypt` funkciju un saglabās jaunu failu.
- Atšifrēšanas process ir apgriezts šifrēšanas procesam.
