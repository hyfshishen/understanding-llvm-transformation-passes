``-partial-inliner``: Partial Inliner
=====

Description
--------
假装这是一段解释。

Code Example
--------

假装这是一段example。

.. code-block:: C

    // original loop
    int x;
    for (x = 0; x < 100; x++)
    {
        delete(x);
    }

    // loop after unrolling (stride size = 4)
    int x; 
    for (x = 0; x < 100; x += 5 )
    {
        delete(x);
        delete(x + 1);
        delete(x + 2);
        delete(x + 3);
        delete(x + 4);
    }
